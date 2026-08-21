# Dapper

**Dapper** é uma biblioteca do ecossistema .NET usada para acessar [[Banco de dados|bancos de dados]] relacionais. Ela é conhecida como um **micro-ORM** (*micro Object-Relational Mapper*), porque ajuda a transformar linhas do banco em objetos da aplicação sem esconder o SQL que a pessoa desenvolvedora escreveu.

Dapper não faz parte da stack principal deste repositório, que usa [[Java]], [[Quarkus]], [[Maven]] e [[PostgreSQL]]. Ele é um assunto relacionado porque mostra outra forma de construir o acesso a dados em um [[Backend]].

## Uma analogia

Imagine que o banco de dados seja uma biblioteca de livros.

- usando acesso direto, você precisa pedir cada livro, abrir a caixa e copiar os dados manualmente;
- usando um ORM completo, um bibliotecário pode procurar, organizar, acompanhar e até atualizar muitos livros por você;
- usando Dapper, você ainda escolhe exatamente quais livros quer buscar, mas recebe ajuda para transformar as informações encontradas em objetos da aplicação.

Dapper reduz o trabalho repetitivo, mas deixa a pessoa no controle do SQL. Ele é uma ferramenta leve entre o código C# e o banco de dados.

## O que é um micro-ORM?

Um ORM completo tenta representar tabelas como objetos e pode oferecer recursos como rastreamento de alterações, relacionamentos automáticos, migrações e consultas construídas pela própria biblioteca.

Um micro-ORM normalmente faz menos coisas:

- executa o SQL informado pela pessoa desenvolvedora;
- envia parâmetros com segurança;
- transforma linhas retornadas em objetos;
- ajuda a executar comandos de inserção, alteração e exclusão;
- oferece versões assíncronas (*async*).

Essa abordagem costuma dar mais controle e previsibilidade, mas exige que a equipe escreva, revise e mantenha mais SQL.

## Como o Dapper funciona

Dapper adiciona métodos de extensão a conexões ADO.NET, como `IDbConnection`. Os métodos mais conhecidos incluem:

- `Query<T>` e `QueryAsync<T>`: consultam várias linhas e as transformam em objetos `T`;
- `QuerySingle<T>` e `QuerySingleAsync<T>`: esperam exatamente uma linha;
- `QuerySingleOrDefault<T>`: aceita uma linha ou nenhuma;
- `Execute` e `ExecuteAsync`: executam comandos que alteram dados e retornam a quantidade de linhas afetadas;
- `ExecuteScalar<T>`: obtém um único valor, como o ID gerado de um registro.

O Dapper não abre uma conexão especial com o banco. A aplicação ainda precisa usar um provedor ADO.NET, como `Npgsql` para PostgreSQL, cuidar da conexão e escolher o momento de iniciar ou confirmar uma transação.

## Instalação

Em um projeto .NET, os pacotes podem ser adicionados com o comando `dotnet`:

```bash
dotnet add package Dapper
dotnet add package Npgsql
```

`Dapper` faz o mapeamento e `Npgsql` permite que uma aplicação .NET converse com o PostgreSQL. Os dois têm responsabilidades diferentes.

## Exemplo de consulta

Considere uma classe simples:

```csharp
public sealed class Usuario
{
    public int Id { get; init; }
    public string Nome { get; init; } = "";
    public string Email { get; init; } = "";
}
```

Uma consulta parametrizada pode ser escrita assim:

```csharp
using Dapper;
using Npgsql;

const string sql = """
    SELECT id, nome, email
    FROM usuarios
    WHERE id = @Id;
    """;

await using var connection = new NpgsqlConnection(connectionString);
await connection.OpenAsync();

var usuario = await connection.QuerySingleOrDefaultAsync<Usuario>(
    sql,
    new { Id = usuarioId });
```

Nesse exemplo:

- `NpgsqlConnection` representa a conexão com o PostgreSQL;
- `@Id` é um parâmetro da consulta;
- `new { Id = usuarioId }` fornece o valor do parâmetro;
- Dapper transforma as colunas `id`, `nome` e `email` em um objeto `Usuario`;
- `QuerySingleOrDefaultAsync` retorna um usuário ou `null` quando não existe resultado.

O nome das colunas normalmente é associado ao nome das propriedades. Quando os nomes forem diferentes, use um alias:

```sql
SELECT nome_completo AS Nome
FROM usuarios;
```

## Inserir, alterar e excluir

Para executar um comando que altera dados, use `ExecuteAsync`:

```csharp
const string sql = """
    INSERT INTO usuarios (nome, email)
    VALUES (@Nome, @Email);
    """;

var linhasAfetadas = await connection.ExecuteAsync(
    sql,
    new { Nome = "Ana", Email = "ana@example.com" });
```

O valor de `linhasAfetadas` informa quantas linhas foram alteradas. Os valores são enviados como parâmetros, e não concatenados dentro do SQL.

Para obter um valor retornado pelo banco, como um ID:

```csharp
const string sql = """
    INSERT INTO usuarios (nome, email)
    VALUES (@Nome, @Email)
    RETURNING id;
    """;

var novoId = await connection.ExecuteScalarAsync<int>(
    sql,
    new { Nome = "Bia", Email = "bia@example.com" });
```

## Transações

Uma transação agrupa operações que precisam ser confirmadas ou desfeitas juntas. Por exemplo, criar um pedido e seus itens normalmente não deve deixar apenas metade dos dados gravada.

```csharp
await using var transaction = await connection.BeginTransactionAsync();

try
{
    await connection.ExecuteAsync(
        "INSERT INTO pedidos (usuario_id) VALUES (@UsuarioId);",
        new { UsuarioId = usuarioId },
        transaction);

    await connection.ExecuteAsync(
        "INSERT INTO itens_pedido (pedido_id, produto_id) VALUES (@PedidoId, @ProdutoId);",
        new { PedidoId = pedidoId, ProdutoId = produtoId },
        transaction);

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

O exemplo é ilustrativo: em uma aplicação real, o ID do pedido deve ser obtido corretamente e todas as operações relacionadas devem usar a mesma conexão e transação.

## Dapper e [[Segurança]]

A principal regra de segurança é usar parâmetros:

```csharp
const string sql = "SELECT id, nome FROM usuarios WHERE email = @Email";

var usuario = await connection.QuerySingleOrDefaultAsync<Usuario>(
    sql,
    new { Email = email });
```

Não faça isto:

```csharp
// Evite: o valor recebido pode alterar o comando SQL.
var sql = $"SELECT id, nome FROM usuarios WHERE email = '{email}'";
```

A concatenação pode permitir **SQL Injection**, um ataque em que a entrada da pessoa é interpretada como parte do comando. Os parâmetros separam os dados da instrução SQL, mas a aplicação ainda deve validar formato, tamanho e regras de negócio no [[Backend]].

Outros cuidados importantes:

- use uma conta do banco com somente as permissões necessárias;
- mantenha a string de conexão fora do código e do [[GitHub]];
- use HTTPS entre o [[Frontend]] e o backend;
- não registre senhas, strings de conexão ou dados sensíveis nos logs;
- defina timeouts e trate cancelamento em consultas demoradas;
- selecione somente as colunas necessárias;
- aplique paginação em listas grandes;
- crie índices adequados e analise consultas lentas;
- valide os dados antes de persistir, sem confiar apenas no cliente.

## Dapper, ADO.NET e ORM completo

| Opção | Como trabalha | Controle do SQL | Abstração |
| --- | --- | --- | --- |
| ADO.NET | acesso de baixo nível ao banco | muito alto | baixa |
| Dapper | executa SQL e mapeia resultados | alto | baixa a média |
| ORM completo, como Entity Framework Core | representa entidades e gerencia mais partes do acesso a dados | médio ou menor, conforme o uso | alta |

Dapper não é necessariamente “melhor” que um ORM completo. Ele pode ser uma boa escolha quando a equipe quer SQL explícito, consultas previsíveis e pouca mágica. Um ORM completo pode ser mais conveniente quando o projeto precisa de rastreamento de entidades, relacionamentos e muitas operações padronizadas.

Na stack Java deste repositório, a comparação aproximada é:

- Dapper se aproxima da ideia de uma biblioteca leve sobre o acesso SQL, como o uso direto de JDBC com mapeamento auxiliar;
- JPA e Hibernate oferecem uma abstração de ORM mais completa;
- [[Quarkus]] pode ser usado com diferentes estratégias de persistência, conforme as necessidades do projeto.

As tecnologias não são intercambiáveis: Dapper pertence ao .NET, enquanto a stack principal usa [[Java]]. A comparação serve para entender as escolhas de arquitetura.

## Dapper e o fluxo da aplicação

Um caminho comum é:

1. o [[Frontend]] envia uma [[Requisição]] para a API;
2. o [[Backend]] autentica, autoriza e valida os dados;
3. uma camada de acesso usa Dapper para executar SQL parametrizado;
4. o PostgreSQL consulta ou altera os dados;
5. o backend devolve uma resposta, normalmente em [[JSON]].

Dapper resolve principalmente o passo 3. Ele não substitui a API, as regras de negócio, a autenticação, o controle de acesso ou o banco de dados.

## Boas práticas

- prefira consultas parametrizadas sempre;
- mantenha SQL e regras de negócio organizados em camadas separadas;
- escolha o método de consulta que representa a quantidade esperada de resultados;
- use `async` para não bloquear threads durante espera de I/O;
- abra a conexão perto da operação e descarte-a ao terminar, aproveitando o pooling do provedor;
- use transações somente quando várias operações precisarem ser atômicas;
- não carregue listas sem limite;
- selecione as colunas necessárias em vez de usar `SELECT *` em código de produção;
- teste consultas, mapeamentos, erros e transações;
- monitore tempo de execução e quantidade de linhas retornadas;
- mantenha Dapper e o provedor do banco atualizados com cuidado.

**Dapper é uma biblioteca leve que reduz o código repetitivo entre C# e o banco de dados, mantendo o SQL sob controle da pessoa desenvolvedora.**
