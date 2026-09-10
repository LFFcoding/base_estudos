# O que é C#?

**C#** (pronuncia-se “C sharp”) é uma linguagem de programação criada pela Microsoft. Ela é usada para criar aplicações de vários tipos, como programas de computador, APIs, jogos, serviços e aplicações para a web.

O C# normalmente é executado sobre o **.NET**, uma plataforma que fornece o ambiente de execução, bibliotecas e ferramentas para desenvolver e executar programas. Uma analogia simples:

- C# é o idioma usado para escrever as instruções;
- .NET é a cozinha, com os utensílios e o ambiente necessários para preparar e executar essas instruções.

C# é uma linguagem de tipagem forte: o compilador verifica os tipos dos dados e ajuda a encontrar muitos erros antes de o programa ser executado. Ele também suporta vários estilos de programação, incluindo programação orientada a objetos, funcional e assíncrona.

## Para que serve?

Com C#, é possível criar:

- aplicações de linha de comando;
- aplicações desktop;
- APIs e sistemas de [[Backend]] com ASP.NET Core;
- serviços que conversam com bancos de dados, como [[PostgreSQL]];
- jogos e ferramentas interativas com a [[Unity Engine]];
- bibliotecas reutilizáveis;
- aplicações que fazem e respondem a [[Requisição]] HTTP.

O C# pertence a um ecossistema diferente da stack principal deste repositório, que usa [[Java]] com [[Quarkus]]. As duas linguagens têm várias ideias parecidas, mas usam ferramentas, bibliotecas e convenções próprias.

## Primeiro programa

Um programa mínimo pode escrever uma mensagem no console:

```csharp
Console.WriteLine("Olá, mundo!");
```

`Console.WriteLine` é um método que escreve uma linha de texto. Em versões modernas do C#, não é obrigatório criar manualmente uma classe `Program` para esse exemplo: o código no arquivo pode ser o ponto inicial da aplicação.

## Tipos e variáveis

Uma variável é como uma caixa com um nome. O tipo define que espécie de valor pode ser guardada nela:

```csharp
string nome = "Ana";
int idade = 30;
decimal saldo = 125.50m;
bool ativo = true;

var cidade = "São Paulo";
```

Nesse exemplo:

- `string` representa texto;
- `int` representa números inteiros;
- `decimal` é adequado para valores financeiros, pois reduz problemas de arredondamento comuns em tipos de ponto flutuante;
- `bool` representa verdadeiro ou falso;
- `var` permite que o compilador descubra o tipo a partir do valor, mas a variável continua tendo um tipo definido.

O C# possui tipos de valor, como `int`, `decimal` e `bool`, e tipos de referência, como classes e `string`. Em termos simples, um tipo de valor guarda o valor diretamente; um tipo de referência aponta para um objeto que está na memória.

Quando um valor pode não existir, é possível indicar isso explicitamente:

```csharp
string? apelido = null;
int? pontuacao = null;
```

O `?` ajuda o compilador a alertar quando o código tenta usar um valor que pode ser `null` sem verificar antes. Ativar e respeitar os nullable reference types é uma boa prática porque evita muitos erros de referência nula.

## Condições e repetições

O programa pode tomar decisões com `if`:

```csharp
if (idade >= 18)
{
    Console.WriteLine("Maior de idade");
}
else
{
    Console.WriteLine("Menor de idade");
}
```

Para repetir uma ação, podem ser usados `for` e `foreach`:

```csharp
for (int i = 0; i < 3; i++)
{
    Console.WriteLine($"Tentativa {i + 1}");
}

var frutas = new[] { "maçã", "banana", "uva" };
foreach (var fruta in frutas)
{
    Console.WriteLine(fruta);
}
```

O `switch` é útil quando existem várias alternativas para o mesmo valor:

```csharp
string status = "aprovado";

string mensagem = status switch
{
    "aprovado" => "Pode continuar",
    "pendente" => "Aguardando análise",
    _ => "Status desconhecido"
};
```

## Métodos

Um método é um bloco nomeado de código que realiza uma tarefa. Ele pode receber parâmetros e devolver um resultado:

```csharp
static decimal CalcularTotal(decimal preco, int quantidade)
{
    return preco * quantidade;
}

decimal total = CalcularTotal(12.50m, 2);
```

Métodos pequenos e com uma responsabilidade clara são mais fáceis de ler, testar e reutilizar. Nomes de métodos em C# normalmente usam PascalCase, como `CalcularTotal`.

## Classes e objetos

Uma classe é um molde; um objeto é algo criado a partir desse molde. Se a classe fosse a planta de uma casa, cada objeto seria uma casa construída com aquela planta.

```csharp
public sealed class Pedido
{
    public long Id { get; init; }
    public decimal Total { get; private set; }

    public Pedido(long id)
    {
        Id = id;
    }

    public void Adicionar(decimal valor)
    {
        if (valor <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(valor));
        }

        Total += valor;
    }
}

var pedido = new Pedido(1);
pedido.Adicionar(25.90m);
```

Nesse exemplo:

- `Id` e `Total` são propriedades;
- o construtor recebe o identificador inicial;
- `private set` impede que qualquer código altere `Total` diretamente;
- `init` permite definir `Id` durante a criação, mas não alterá-lo depois;
- `sealed` impede que outra classe herde de `Pedido` quando essa extensão não for desejada;
- `throw` interrompe a operação porque o valor recebido é inválido.

Encapsular as mudanças dentro de métodos, em vez de deixar todos os campos públicos, protege as regras do objeto. Essa preocupação se relaciona aos princípios [[SOLID]] e a outros [[Design Pattern]].

## Interfaces, enums, records e structs

Uma interface descreve um contrato: ela informa quais operações uma classe deve oferecer, sem obrigar a implementação a ser igual.

```csharp
public interface INotificador
{
    Task EnviarAsync(string mensagem, CancellationToken cancellationToken);
}
```

Qualquer classe que implemente `INotificador` precisa fornecer `EnviarAsync`. Isso facilita trocar implementações e escrever testes com objetos falsos (*mocks* ou *fakes*). A ideia é parecida com separar uma dependência por contrato, como discutido em [[Chamada estática vs injeção CDI]].

Um `enum` representa um conjunto pequeno e conhecido de opções:

```csharp
public enum StatusPedido
{
    Criado,
    Pago,
    Enviado,
    Cancelado
}
```

Um `record` é útil para dados cujo conteúdo é mais importante que uma identidade individual. Ele oferece comparação por valor de forma conveniente:

```csharp
public record Endereco(string Cidade, string Estado);
```

Um `struct` é um tipo de valor, adequado para objetos pequenos cujo valor completo pode ser copiado. Não é uma boa prática transformar qualquer classe em `struct` sem avaliar o tamanho e o comportamento de cópia.

## Coleções e generics

Coleções armazenam vários valores. `List<T>` representa uma lista e `Dictionary<TKey, TValue>` associa uma chave a um valor:

```csharp
var ids = new List<int> { 10, 20, 30 };
var usuariosPorId = new Dictionary<int, string>
{
    [1] = "Ana",
    [2] = "Bruno"
};
```

O `T` em `List<T>` é um exemplo de *generic*: a mesma estrutura pode trabalhar com diferentes tipos mantendo a segurança da tipagem. Prefira coleções genéricas a coleções que armazenam tudo como `object`, porque o compilador consegue detectar mais erros.

## LINQ

LINQ (*Language Integrated Query*) permite filtrar, ordenar, transformar e agrupar coleções usando uma sintaxe integrada ao C#. Ele é parecido com fazer perguntas organizadas a uma lista.

```csharp
var nomes = usuarios
    .Where(usuario => usuario.Ativo)
    .OrderBy(usuario => usuario.Nome)
    .Select(usuario => usuario.Nome)
    .ToList();
```

Nesse fluxo:

1. `Where` mantém somente usuários ativos;
2. `OrderBy` ordena pelo nome;
3. `Select` transforma cada usuário em seu nome;
4. `ToList` materializa o resultado em uma lista.

`OrderBy` e `ThenBy` são formas comuns de aplicar [[Sorting]]. Muitas operações LINQ usam execução adiada (*deferred execution*): a consulta é montada primeiro e executada quando o resultado é percorrido. Use `ToList`, `ToArray` ou outra operação final quando precisar capturar o resultado naquele momento.

Boas práticas para LINQ:

- filtre antes de projetar ou ordenar quando isso reduzir a quantidade de dados;
- evite consultas longas demais; extraia partes complexas para métodos nomeados;
- tome cuidado ao executar a mesma consulta várias vezes;
- ao consultar um banco de dados, confirme quais operações serão convertidas para SQL e quais serão executadas na aplicação.

## Programação assíncrona

O C# usa `Task`, `async` e `await` para operações que podem demorar, como chamadas de rede ou leitura de arquivos. Enquanto espera a resposta, a aplicação pode liberar o fluxo para realizar outro trabalho.

```csharp
public static async Task<string> BuscarAsync(
    HttpClient client,
    CancellationToken cancellationToken)
{
    using var response = await client.GetAsync(
        "https://api.exemplo.com/usuarios",
        cancellationToken);

    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync(cancellationToken);
}
```

O `CancellationToken` permite cancelar uma operação que deixou de ser necessária. `EnsureSuccessStatusCode` transforma uma resposta HTTP de erro em uma exceção, evitando tratar uma resposta malsucedida como se fosse válida.

Boas práticas:

- propague `async` até o ponto em que a operação começa;
- use `await` em vez de bloquear com `.Result` ou `.Wait()`;
- passe `CancellationToken` em operações que podem demorar;
- evite `async void`, exceto em manipuladores de eventos que exigem esse formato;
- não crie tarefas sem observar seus erros ou sem saber quem é responsável por aguardá-las.

## Erros e exceções

Exceções representam situações anormais, como um arquivo que não existe ou uma entrada inválida:

```csharp
try
{
    pedido.Adicionar(valorRecebido);
}
catch (ArgumentOutOfRangeException erro)
{
    Console.WriteLine($"Valor inválido: {erro.Message}");
}
```

Capture somente as exceções que você sabe tratar. Um `catch (Exception)` genérico que apenas esconde o erro pode dificultar a investigação de problemas. Registre informações úteis sem expor senhas, tokens ou dados pessoais.

O C# possui coletor de lixo (*garbage collector*), que libera objetos que não são mais usados. Isso não elimina a necessidade de liberar recursos externos, como arquivos, conexões e respostas HTTP. Para esses recursos, use `using` ou `await using`:

```csharp
using var arquivo = File.OpenRead("dados.txt");
// O arquivo será fechado automaticamente ao sair do escopo.
```

## Projetos e ferramentas do .NET

O SDK do .NET fornece a ferramenta `dotnet`, usada para criar, compilar e executar projetos:

```bash
dotnet new console -n ExemploCSharp
cd ExemploCSharp
dotnet run
dotnet build
dotnet test
```

O comando `dotnet new` cria um projeto a partir de um modelo. `dotnet run` compila e executa, `dotnet build` apenas compila e `dotnet test` executa os testes configurados. Esses comandos são usados no [[Terminal]].

O arquivo `.csproj` descreve o projeto, suas configurações e suas dependências. Pacotes de terceiros normalmente são obtidos pelo NuGet, o gerenciador de pacotes do ecossistema .NET. Nesse ponto existe uma ideia semelhante ao `pom.xml` e ao [[Maven]] no ecossistema Java, embora as ferramentas e os formatos sejam diferentes.

## C# no backend e em APIs

C# pode ser usado no [[Backend]] para criar APIs, autenticar usuários, validar dados e acessar bancos de dados. Uma API pode receber uma [[Requisição]] HTTP, executar uma regra de negócio e devolver dados em [[JSON]].

Em aplicações de produção:

- valide os dados recebidos na borda da aplicação;
- não confie em valores enviados pelo cliente;
- mantenha credenciais e chaves fora do código-fonte;
- use logs sem registrar segredos ou dados sensíveis;
- aplique autenticação e autorização de acordo com a operação;
- escreva [[Testes]] para regras importantes;
- use analisadores estáticos e ferramentas de [[Linting]] no processo de desenvolvimento;
- versione o código com [[Git]].

## C# na Unity

A [[Unity Engine]] usa C# para os scripts que controlam o comportamento dos objetos do jogo. Um script pode herdar de `MonoBehaviour` para ser associado a um GameObject e participar do ciclo de vida da cena:

```csharp
using UnityEngine;

public class Mover : MonoBehaviour
{
    [SerializeField] private float velocidade = 5f;

    private void Update()
    {
        transform.Translate(
            Vector3.forward * velocidade * Time.deltaTime);
    }
}
```

`Update` é chamado pela Unity durante o jogo. `Time.deltaTime` ajuda a fazer o movimento depender do tempo, e não apenas da quantidade de quadros por segundo. A mesma linguagem pode ser usada para implementar um [[Mapa procedural]], regras de jogo, interfaces e ferramentas do editor.

## C# e Java: semelhanças e diferenças

As duas linguagens:

- são compiladas para um ambiente de execução;
- possuem tipagem forte;
- suportam classes, interfaces, generics e exceções;
- possuem coleta automática de lixo;
- são usadas em aplicações de backend.

Algumas diferenças importantes:

- C# é executado principalmente no .NET; Java é executado na JVM;
- C# usa arquivos de projeto `.csproj` e NuGet; Java frequentemente usa Maven ou Gradle;
- C# possui recursos como propriedades, LINQ e `async`/`await` integrados à linguagem e ao ecossistema;
- Java e C# têm sintaxes parecidas, mas uma não é uma versão da outra.

Conhecer uma ajuda a aprender a outra, mas os detalhes de bibliotecas, ferramentas e convenções ainda precisam ser estudados separadamente.

## Boas práticas resumidas

- habilite as verificações de valores nulos e trate `null` de forma explícita;
- use `decimal` para dinheiro e valide limites de entrada;
- mantenha classes e métodos com responsabilidades pequenas;
- prefira interfaces quando a troca de implementação for uma necessidade real;
- encapsule o estado e proteja as regras do domínio;
- use `async`/`await` corretamente e permita cancelamento;
- descarte recursos com `using`;
- não coloque segredos no código, no repositório ou nos logs;
- escreva testes para comportamentos importantes;
- mantenha formatação e análise estática automatizadas;
- escolha nomes claros e evite abreviações que dificultem a leitura.

## Em uma frase

**C# é uma linguagem de programação versátil e fortemente tipada, normalmente usada com o .NET para criar aplicações, APIs, serviços e jogos, inclusive na Unity.**

## Fontes

- [Guia oficial do C# — Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/)
- [Sistema de tipos do C# — Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/types/)
- [LINQ — Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/linq/)
- [Programação assíncrona — Microsoft](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
- [Introdução ao .NET — Microsoft](https://learn.microsoft.com/en-us/dotnet/core/introduction)
- [MonoBehaviour — Unity](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html)
