# O que significa `final`?

`final` é uma palavra-chave usada em [[Java]] para impedir determinadas mudanças. Dependendo do lugar onde aparece, ela pode impedir que uma variável receba outro valor, que um método seja sobrescrito ou que uma classe seja estendida.

Uma analogia: `final` coloca um aviso de “não altere esta parte”. O aviso pode proteger uma etiqueta, uma regra de funcionamento ou a própria estrutura do objeto. O significado exato depende do elemento marcado.

`final` não significa sempre “imutável” e não possui o mesmo uso em todas as linguagens:

- em Java, `final` pode ser usado em variáveis, métodos e classes;
- em C#, conceitos parecidos usam `readonly`, `const` e `sealed`;
- em C++, `final` impede herança ou sobrescrita, enquanto `const` trata de não modificar valores por uma expressão.

## `final` em variáveis Java

Uma variável `final` só pode receber uma atribuição:

```java
public final int LIMITE_TENTATIVAS = 3;

final int idadeMinima = 18;
// idadeMinima = 21; // Não compila.
```

Uma variável local `final` também pode ser inicializada depois, desde que receba valor uma única vez:

```java
final String ambiente;

if (producao) {
    ambiente = "produção";
} else {
    ambiente = "desenvolvimento";
}
```

O compilador garante que todos os caminhos válidos atribuam um valor e que nenhum caminho tente atribuir novamente.

## Referência final não torna o objeto imutável

Este ponto é muito importante:

```java
final List<String> nomes = new ArrayList<>();
nomes.add("Ana");

// nomes = new ArrayList<>(); // Não compila.
```

`final` impede trocar a referência para outra lista, mas não impede alterar o conteúdo da lista. Para obter imutabilidade, o objeto também precisa não oferecer operações de alteração ou deve ser protegido por uma visão imutável:

```java
final List<String> nomes =
        List.of("Ana", "Bruno");
```

Uma referência final, um objeto imutável e uma coleção não modificável são ideias relacionadas, mas não são sinônimos.

## `final` em campos

Campos `final` precisam ser inicializados no ponto de declaração, em um bloco de inicialização ou no construtor:

```java
public final class Pedido {
    private final long id;
    private final String cliente;

    public Pedido(long id, String cliente) {
        this.id = id;
        this.cliente = cliente;
    }

    public long id() {
        return id;
    }
}
```

Depois que o construtor termina, os campos não podem receber outra referência. Isso torna o estado mais previsível e pode ajudar a criar objetos imutáveis.

Para que a classe seja realmente imutável, também é necessário:

- não expor objetos internos mutáveis diretamente;
- copiar valores mutáveis recebidos;
- retornar cópias ou visões não modificáveis;
- garantir que subclasses não alterem o comportamento, normalmente usando `final` na classe;
- validar o estado no construtor.

## `final` em métodos

Um método `final` não pode ser sobrescrito por uma subclasse:

```java
public abstract class Relatorio {
    public final String gerar() {
        validar();
        return montarConteudo();
    }

    protected abstract String montarConteudo();

    private void validar() {
        // Validação comum.
    }
}
```

A classe-base controla `gerar`, mas permite que subclasses implementem `montarConteudo`. Essa combinação aparece no padrão Template Method: o fluxo é fixo e algumas etapas são variáveis.

Use `final` quando quebrar ou alterar esse fluxo tornaria a classe incorreta. Não marque todos os métodos como finais sem avaliar se a extensão é realmente necessária.

## `final` em classes

Uma classe `final` não pode ser estendida:

```java
public final class Identificador {
    private final String valor;

    public Identificador(String valor) {
        this.valor = valor;
    }
}
```

Isso comunica que a classe não foi projetada para herança e impede subclasses acidentais. Classes de valor, objetos imutáveis e componentes cujo comportamento não deve ser alterado podem se beneficiar disso.

Uma classe não pode ser simultaneamente `abstract` e `final` em Java: `abstract` exige que outras classes possam estendê-la, enquanto `final` proíbe extensão.

## `final` e herança

`final` define uma fronteira de extensão:

```text
classe-base
├── método final: não pode ser sobrescrito
├── método normal: pode seguir as regras da linguagem
└── classe final: não aceita subclasses
```

Uma subclasse deve respeitar o contrato da base. Se nenhum código deve substituir o comportamento, `final` torna essa intenção verificável pelo compilador.

O assunto se relaciona a [[Extends]], [[Classe abstrata]] e [[Polimorfismo]].

## `final` em parâmetros

Java permite marcar parâmetros como `final`:

```java
public void enviar(final String mensagem) {
    // mensagem = "outra"; // Não compila.
}
```

Isso impede reatribuir o parâmetro dentro do método. Não torna um objeto passado como parâmetro imutável e nem costuma ser necessário em todo método. Use quando a restrição melhorar a clareza ou for exigida por uma convenção da equipe.

## `final` em C#

Em [[C#]], não existe uma palavra-chave `final` com exatamente o mesmo uso. Os equivalentes dependem do objetivo:

| Objetivo | C# |
|---|---|
| impedir que uma classe seja herdada | `sealed` |
| impedir alteração de um campo depois da construção | `readonly` |
| declarar uma constante conhecida em compilação | `const` |
| permitir definir uma propriedade somente na criação | `init` |
| impedir sobrescrita de um membro virtual | `sealed override` |

Exemplo:

```csharp
public sealed class Pedido
{
    private readonly long _id;

    public Pedido(long id)
    {
        _id = id;
    }

    public long Id => _id;
}
```

`readonly` impede atribuições ao campo fora do construtor, mas um campo readonly que aponta para um objeto mutável ainda pode apontar para um objeto cujo conteúdo muda. A mesma diferença entre referência e objeto continua existindo.

## `final` em C++

Em [[C++]], `final` impede que uma classe seja herdada ou que um método virtual seja sobrescrito:

```cpp
class Base
{
public:
    virtual ~Base() = default;
    virtual void executar() = 0;
};

class Implementacao final : public Base
{
public:
    void executar() override
    {
    }
};
```

Também é possível marcar um método:

```cpp
class Especializacao : public Base
{
public:
    void executar() final override
    {
    }
};
```

Depois disso, classes derivadas de `Especializacao` não podem sobrescrever `executar`.

Em C++, `const` é usado para indicar que um valor ou método não deve modificar algo por aquela expressão. `const` e `final` têm objetivos diferentes:

- `const`: restringe modificação de valores ou estado observado;
- `final`: restringe herança ou sobrescrita.

## `final` não é segurança

Marcar um campo, método ou classe como `final` não protege um sistema contra usuários mal-intencionados. A palavra-chave ajuda o compilador e organiza o design, mas não substitui:

- autenticação;
- autorização;
- criptografia;
- validação de entradas;
- proteção de segredos;
- controles de acesso no [[Backend]].

Dados finais ainda podem ser enviados em uma [[Requisição]] ou expostos em [[JSON]]. As regras de [[Segurança]] continuam necessárias.

## `final` e injeção de dependência

Campos finais combinam bem com injeção por construtor:

```java
public final class PedidoService {
    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
```

Depois que o serviço é construído, sua dependência não pode ser trocada acidentalmente. Isso torna o objeto mais previsível e facilita compreender suas dependências.

Em frameworks como [[Quarkus]], a injeção precisa respeitar a forma de construção e o ciclo de vida configurados. Não use `final` sem verificar se a ferramenta e a forma de injeção escolhida são compatíveis.

## `final` e testes

`final` pode dificultar a substituição de classes por herança em alguns frameworks de mock. Uma boa alternativa é depender de interfaces e injetar fakes:

```java
public interface Relogio {
    Instant agora();
}

public final class Sistema {
    private final Relogio relogio;

    public Sistema(Relogio relogio) {
        this.relogio = relogio;
    }
}
```

O uso de uma classe final não deve ser removido apenas para permitir um teste mal estruturado. Prefira contratos e composição quando existir uma dependência que precise ser substituída.

Isso se relaciona a [[Testes]], [[Interface abstrata]] e [[Polimorfismo]].

## `final` versus `finally`

Em Java, `final` e `finally` são palavras diferentes:

- `final` restringe alteração, sobrescrita ou herança;
- `finally` define um bloco que normalmente será executado depois de um `try` e seus tratamentos;
- `finalize` foi um mecanismo antigo de finalização e não deve ser confundido com `final`.

Exemplo de `finally`:

```java
try {
    executarOperacao();
} finally {
    liberarRecurso();
}
```

Para recursos modernos, prefira construções apropriadas, como `try-with-resources`, em vez de depender de finalização automática.

## Relação com padrões de projeto

`final` pode apoiar decisões de vários [[Design Pattern]]:

- em [[Factory Pattern]], uma factory pode retornar objetos finais quando não há extensão suportada;
- em [[Strategy Pattern]], estratégias podem ser finais quando seu comportamento não deve ser herdado;
- em uma classe abstrata, métodos finais podem proteger o fluxo comum;
- em objetos de valor, campos finais ajudam a preservar o estado.

A palavra-chave não cria um padrão sozinha. Ela apenas reforça uma decisão de extensão ou mutabilidade.

## Boas práticas

- use `final` quando a restrição expressar uma regra real;
- use campos finais para dependências definidas no construtor;
- não confunda referência final com objeto imutável;
- proteja coleções internas contra alterações externas;
- use `final` em classes que não foram projetadas para herança;
- use `final` em métodos quando o fluxo-base não puder ser alterado;
- prefira composição e interfaces para variações de comportamento;
- combine `final` com validação e invariantes;
- avalie o impacto em frameworks, proxies e ferramentas de teste;
- use `sealed`, `readonly`, `const` e `final` de acordo com a linguagem;
- documente por que uma extensão ou alteração foi proibida;
- teste o comportamento público importante.

## Em uma frase

**`final` restringe alterações, sobrescritas ou heranças conforme o elemento marcado; ele ajuda a proteger o design, mas não torna automaticamente um objeto imutável ou seguro.**

### Veja também

- [[Public]]
- [[Private]]
- [[Protected]]
- [[Classe abstrata]]
- [[Interface abstrata]]
- [[Implements]]
- [[Extends]]
- [[Polimorfismo]]
- [[SOLID]]
- [[Design Pattern]]
- [[Factory Pattern]]
- [[Strategy Pattern]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
- [[Segurança]]
