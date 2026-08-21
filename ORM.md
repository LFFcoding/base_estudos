# ORM

**ORM** significa *Object-Relational Mapping* (mapeamento objeto-relacional). É uma técnica e um conjunto de ferramentas que conectam objetos do código a tabelas e registros de um banco de dados relacional.

Uma analogia: o banco fala em tabelas, linhas e colunas; o programa fala em classes, objetos e propriedades. O ORM funciona como um tradutor entre esses dois idiomas.

O ORM pode gerar consultas, transformar resultados em objetos e acompanhar relações. Ele reduz código repetitivo, mas não elimina a necessidade de entender [[PostgreSQL]], SQL, índices, transações e modelagem.

## O problema que o ORM tenta resolver

Em um banco relacional:

- dados ficam em tabelas;
- cada registro ocupa uma linha;
- cada atributo ocupa uma coluna;
- relações usam chaves estrangeiras;
- consultas são escritas em SQL.

Em um programa orientado a objetos:

- dados ficam em objetos;
- objetos possuem propriedades;
- objetos podem conter outros objetos;
- relações aparecem como referências ou coleções;
- comportamento fica junto da classe.

Essa diferença é chamada de **impedance mismatch** (incompatibilidade entre o modelo de objetos e o modelo relacional). O ORM tenta reduzir o trabalho de converter um modelo para o outro.

## Mapeamento básico

| Código | Banco relacional |
| --- | --- |
| classe ou entidade | tabela |
| objeto | linha |
| propriedade | coluna |
| identificador do objeto | chave primária |
| referência a outro objeto | chave estrangeira |
| coleção de objetos | relação entre tabelas |
| método de consulta | SQL gerado ou executado |

Esse mapa é uma aproximação. Uma classe pode ser dividida em várias tabelas, e uma tabela pode ser lida por mais de um modelo do código.

## Exemplo com Java

No ecossistema [[Java]], uma entidade pode ser mapeada com anotações de JPA, geralmente usando Hibernate:

```java
@Entity
@Table(name = "usuarios")
public class Usuario {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String nome;

    @Column(nullable = false, unique = true)
    private String email;
}
```

Esse exemplo indica que `Usuario` será persistido na tabela `usuarios`, que `id` identifica cada registro e que `email` deve ser único.

Em uma aplicação com [[Quarkus]], a equipe pode usar Hibernate ORM e APIs de persistência para salvar e consultar entidades. O [[Maven]] organiza as dependências do projeto.

## Exemplo com Prisma

No ecossistema [[Node.js]] e [[TypeScript]], o [[Prisma]] é uma opção de ORM. O modelo pode ser descrito no schema:

```prisma
model Usuario {
  id    Int    @id @default(autoincrement())
  nome  String
  email String @unique
}
```

Depois, o cliente gerado pode consultar os registros:

```typescript
const usuarios = await prisma.usuario.findMany({
  select: {
    id: true,
    nome: true,
    email: true,
  },
});
```

Hibernate e Prisma são ferramentas diferentes, mas resolvem uma necessidade parecida: aproximar o modelo usado no código do modelo persistido no banco.

## CRUD

ORMs normalmente oferecem operações CRUD:

- **Create:** criar um registro;
- **Read:** consultar registros;
- **Update:** alterar registros;
- **Delete:** excluir registros.

Um fluxo de criação pode ser:

```text
objeto do programa -> ORM -> INSERT -> tabela
```

Um fluxo de leitura pode ser:

```text
SELECT -> ORM -> objeto do programa
```

O ORM pode evitar que a equipe escreva a mesma conversão em várias partes, mas as operações ainda têm custo no banco.

## Entidades e objetos de transferência

Uma entidade representa dados persistidos e costuma ter identidade, como um `id`. Ela não precisa ser igual ao objeto enviado pela API.

É comum separar:

- **entidade:** modelo persistido no banco;
- **DTO:** formato usado na entrada ou saída da API;
- **mapper:** código que converte entidade em DTO e DTO em entidade;
- **repository:** componente responsável por consultar o banco;
- **service:** componente que aplica regras de negócio.

Essa separação evita devolver campos internos, como hashes de senha, e impede que qualquer campo recebido pela API seja salvo sem análise.

## Relações

Um ORM pode representar relações como:

- um para um (*one-to-one*);
- um para muitos (*one-to-many*);
- muitos para um (*many-to-one*);
- muitos para muitos (*many-to-many*).

Exemplo conceitual:

```text
Usuario 1 -------- N Pedido
```

No banco, `Pedido` pode ter uma coluna `usuario_id` apontando para `Usuario.id`. No código, um usuário pode expor uma coleção de pedidos.

Relações facilitam a navegação, mas não devem fazer a aplicação carregar dados que não serão usados. Escolha conscientemente quais relações serão consultadas.

## Lazy e eager loading

- **Lazy loading:** busca a relação somente quando ela é acessada;
- **Eager loading:** busca a relação junto com o objeto principal.

Lazy loading pode economizar dados em alguns fluxos, mas pode disparar consultas escondidas quando o código percorre uma coleção. Eager loading pode simplificar uma tela, mas trazer dados demais.

Não escolha uma estratégia apenas por padrão. Observe as consultas produzidas e o volume de dados.

## Problema N+1

O problema N+1 acontece quando a aplicação faz uma consulta para buscar uma lista e depois uma nova consulta para cada item:

```text
1 consulta para buscar 100 usuários
100 consultas para buscar os pedidos de cada usuário
total: 101 consultas
```

Isso pode deixar uma tela lenta. Soluções possíveis incluem:

- buscar as relações em uma consulta planejada;
- usar `join` ou `include` quando fizer sentido;
- agrupar consultas;
- limitar a quantidade de dados;
- revisar o plano de execução do banco.

O ORM não elimina o N+1 automaticamente. A abstração pode até esconder o problema, por isso monitore o SQL gerado.

## Transações

Uma transação agrupa operações que precisam ser confirmadas ou desfeitas juntas:

```text
iniciar transação
    criar pedido
    atualizar estoque
confirmar tudo
```

Se uma operação falhar, a aplicação pode desfazer as mudanças da transação. Use transações para preservar regras importantes, mas mantenha-as curtas para evitar locks e conexões ocupadas.

Uma transação do ORM não resolve automaticamente a publicação de uma mensagem em um broker. Quando o fluxo envolve banco e mensageria, considere [[Outbox Pattern]].

## Migrations não são o mesmo que ORM

Um ORM cuida principalmente do mapeamento e do acesso aos dados. Uma ferramenta de migration controla a evolução da estrutura do banco.

Alguns produtos juntam as duas funções. O Prisma, por exemplo, possui Prisma Migrate; no ecossistema Java, projetos podem usar migrations SQL junto com Hibernate.

Migrations devem ser versionadas e revisadas. Não altere a estrutura de produção manualmente sem registrar como a mudança será reproduzida em outros ambientes.

## ORM e SQL direto

ORM não significa que SQL deixou de existir. Há três abordagens comuns:

| Abordagem | Característica |
| --- | --- |
| ORM | mais abstração e integração com objetos |
| query builder | monta consultas por uma API, mantendo mais controle |
| SQL direto | controle explícito da consulta e dos recursos do banco |

Use ORM para operações comuns e produtividade. Use SQL ou consultas nativas quando precisar de uma função específica, uma consulta complexa ou uma otimização comprovada.

Sempre passe parâmetros de forma segura. Nunca concatene entrada do usuário diretamente em SQL.

## Vantagens

- reduz código repetitivo de persistência;
- aproxima o banco dos modelos do código;
- fornece APIs reutilizáveis para CRUD;
- pode oferecer tipos, autocomplete e conversões;
- organiza relações e transações;
- facilita a manutenção de operações comuns;
- pode integrar migrations, validações e ferramentas de desenvolvimento.

## Limitações

- a consulta gerada pode ser ineficiente;
- abstrações podem esconder joins e múltiplas consultas;
- recursos específicos do banco podem não estar bem representados;
- migrations automáticas podem ser perigosas sem revisão;
- modelos de objetos não resolvem todos os problemas de modelagem relacional;
- o uso incorreto pode gerar N+1, excesso de dados ou muitas conexões;
- aprender o ORM não elimina a necessidade de aprender SQL.

## ORM não é regra de negócio

O ORM deve cuidar do acesso e da persistência. Regras como “um pedido só pode ser cancelado antes do envio” pertencem ao service ou domínio, não devem ficar escondidas em uma consulta genérica.

Um fluxo organizado de [[Backend]] pode ser:

```text
requisição -> controller/resource -> service -> repository/ORM -> PostgreSQL
```

O controller recebe a entrada, o service aplica regras, o repository consulta e o ORM traduz a operação para o banco.

## Segurança

- valide e autorize operações antes de persistir dados;
- não devolva entidades diretamente sem revisar campos sensíveis;
- use parâmetros, evitando injeção SQL;
- não confie em tipos do código para validar entradas externas;
- mantenha credenciais fora do código e do Git;
- use usuário do banco com permissões mínimas;
- revise migrations destrutivas;
- limite paginação, filtros e tamanho de consultas;
- evite registrar senhas, tokens ou objetos completos nos logs;
- teste autorização, transações, concorrência e restauração;
- combine o ORM com as orientações de [[Segurança]] e [[Testes]].

## Boas práticas

- aprenda SQL antes de depender da abstração;
- modele as tabelas e relações conforme as consultas do sistema;
- selecione somente os campos necessários;
- analise consultas lentas e planos de execução;
- defina índices com base em buscas reais;
- evite carregar relações automaticamente sem necessidade;
- use paginação em listas grandes;
- mantenha transações curtas;
- versione e revise migrations;
- centralize a criação do client ou do EntityManager;
- não crie uma conexão nova para cada requisição;
- use SQL nativo quando houver uma necessidade comprovada;
- escreva testes para regras de negócio e integrações com o banco.

## Resumo

**ORM é uma camada que traduz entre objetos do código e dados relacionais do banco. Ele aumenta a produtividade e organiza operações comuns, mas não substitui SQL, modelagem, segurança, análise de desempenho e testes.**

## Referências oficiais

- [Hibernate ORM](https://hibernate.org/orm/);
- [O que é ORM, segundo o Hibernate](https://hibernate.org/orm/what-is-an-orm/);
- [Visão geral do Prisma ORM](https://docs.prisma.io/docs/orm);
- [Prisma Client](https://www.prisma.io/docs/orm/prisma-client).
