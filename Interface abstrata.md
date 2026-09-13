# O que é uma interface abstrata?

O termo **abstract interface** pode significar coisas diferentes conforme a linguagem. Em geral, ele descreve uma interface que define um contrato sem dizer como cada classe deve implementar esse contrato.

Na maioria das linguagens modernas, uma interface já é abstrata por natureza: ela não representa um objeto completo que possa ser criado diretamente. Ela representa uma capacidade ou conjunto de operações que outras classes devem oferecer.

Uma analogia: uma tomada define o formato e a tensão esperada para que um aparelho possa ser conectado. Ela não define como a televisão ou o carregador funciona por dentro; apenas estabelece um contrato de conexão.

## O ponto mais importante

`abstract interface` não é uma construção igual em Java, C# e C++:

- em **Java**, uma interface já é um tipo abstrato; escrever `abstract interface` é redundante;
- em **C#**, a palavra-chave usada é `interface`; não se escreve `abstract interface`;
- em **C++**, não existe uma palavra-chave padrão `interface`; uma classe abstrata com métodos virtuais puros costuma representar essa ideia.

Portanto, quando alguém diz “interface abstrata”, normalmente está falando de uma interface ou de um contrato abstrato, não necessariamente de uma palavra-chave específica.

## Para que serve uma interface?

Uma interface permite que classes diferentes ofereçam a mesma capacidade:

```text
Pagamento → contrato GatewayPagamento
                 ├── cartão
                 ├── boleto
                 └── Pix
```

O serviço que realiza o pagamento depende do contrato `GatewayPagamento`, não dos detalhes de cada provedor.

Interfaces ajudam a:

- reduzir acoplamento;
- trocar implementações;
- usar polimorfismo;
- facilitar testes com fakes e mocks;
- separar regras de negócio de detalhes externos;
- combinar capacidades em classes diferentes.

O assunto se conecta a [[Polimorfismo]], [[SOLID]] e [[Design Pattern]].

## Interface em Java

Em [[Java]], uma interface pode ser declarada assim:

```java
public interface GatewayPagamento {
    ResultadoPagamento cobrar(BigDecimal valor);
}
```

O método sem corpo representa uma operação que as implementações precisam fornecer. Em uma interface tradicional, os métodos de instância sem implementação são implicitamente públicos e abstratos.

Classes diferentes podem implementar o contrato:

```java
public final class GatewayPix implements GatewayPagamento {
    @Override
    public ResultadoPagamento cobrar(BigDecimal valor) {
        return ResultadoPagamento.aprovado();
    }
}

public final class GatewayCartao implements GatewayPagamento {
    @Override
    public ResultadoPagamento cobrar(BigDecimal valor) {
        return ResultadoPagamento.aprovado();
    }
}
```

Uso polimórfico:

```java
GatewayPagamento gateway = new GatewayPix();
ResultadoPagamento resultado = gateway.cobrar(valor);
```

A variável conhece `GatewayPagamento`, mas o objeto concreto é `GatewayPix`.

### `abstract interface` em Java

Java permite declarar uma interface com `abstract` explicitamente:

```java
public abstract interface GatewayPagamento {
    ResultadoPagamento cobrar(BigDecimal valor);
}
```

Porém, o modificador é redundante. A forma usual e mais clara é escrever apenas `interface`.

### Métodos default

Interfaces Java também podem fornecer uma implementação padrão com `default`:

```java
public interface Identificavel {
    String id();

    default boolean possuiId() {
        return id() != null && !id().isBlank();
    }
}
```

Isso não transforma a interface em uma classe abstrata com estado. O método padrão fornece comportamento, mas a interface continua sem campos de instância e sem construtor de objetos.

Use métodos `default` com cuidado. Eles são úteis para comportamento realmente comum ou para evolução compatível de uma interface, mas podem esconder dependências e regras demais no contrato.

## Interface em C#

Em [[C#]], a declaração normal é:

```csharp
public interface INotificador
{
    void Enviar(string mensagem);
}
```

Uma classe implementa a interface:

```csharp
public sealed class NotificadorEmail : INotificador
{
    public void Enviar(string mensagem)
    {
        Console.WriteLine($"Enviando e-mail: {mensagem}");
    }
}
```

Não se escreve:

```csharp
// Não é a forma normal nem válida de declarar uma interface em C#:
// public abstract interface INotificador { }
```

A própria palavra `interface` define o contrato. C# moderno também permite implementações padrão em membros de interface e membros `static abstract` em cenários genéricos específicos. Esses recursos não significam que uma interface seja uma classe abstrata: interfaces continuam sem estado de instância e sem construtores de objetos.

Por convenção, interfaces C# frequentemente começam com `I`, como `INotificador`. Essa é uma convenção de nomes, não uma exigência da linguagem.

## Interface em C++

Em [[C++]], o padrão da linguagem não possui uma palavra-chave `interface`. Uma interface pode ser representada por uma classe abstrata com métodos virtuais puros:

```cpp
class Notificador
{
public:
    virtual ~Notificador() = default;
    virtual void enviar(const std::string& mensagem) = 0;
};

class NotificadorEmail final : public Notificador
{
public:
    void enviar(const std::string& mensagem) override
    {
        std::cout << "Enviando e-mail: " << mensagem << '\n';
    }
};
```

O `= 0` torna `enviar` uma função virtual pura e faz com que `Notificador` seja abstrata. O destrutor virtual é importante quando objetos derivados podem ser destruídos por meio de um ponteiro para a classe-base.

Essa classe pode conter dados ou métodos concretos, mas uma interface “pura” em C++ normalmente contém apenas operações virtuais públicas e um destrutor virtual.

## Interface versus classe abstrata

Uma interface e uma classe abstrata podem definir contratos, mas não têm exatamente o mesmo papel:

| Interface | Classe abstrata |
|---|---|
| descreve uma capacidade ou contrato | representa uma base comum |
| normalmente não possui estado de instância | pode possuir campos e propriedades |
| não possui construtor de objeto | pode ter construtor |
| uma classe pode implementar várias interfaces | Java e C# permitem uma classe-base principal |
| favorece composição de capacidades | favorece reutilização de estado e comportamento |
| pode ser implementada por tipos sem uma mesma origem | representa uma relação mais forte de “é um” |

Use uma interface quando o contrato for o mais importante. Use uma classe abstrata quando as subclasses realmente compartilharem invariantes, estado ou um fluxo comum. Mais detalhes estão em [[Classe abstrata]].

## Interface versus classe concreta

Uma interface não é uma classe pronta para criar:

```java
// Não compila:
var gateway = new GatewayPagamento();
```

É necessário criar uma implementação concreta:

```java
GatewayPagamento gateway = new GatewayPix();
```

O tipo da variável pode ser a interface, enquanto o objeto criado é uma classe concreta. Essa separação permite trocar a implementação sem alterar todo o código que usa o contrato.

## Interfaces e injeção de dependência

Uma interface é frequentemente usada como ponto de injeção:

```java
public final class PagamentoService {
    private final GatewayPagamento gateway;

    public PagamentoService(GatewayPagamento gateway) {
        this.gateway = gateway;
    }

    public ResultadoPagamento pagar(BigDecimal valor) {
        return gateway.cobrar(valor);
    }
}
```

O serviço não cria `GatewayPix` nem `GatewayCartao`. A implementação é fornecida pelo código de composição ou por um framework. Isso se relaciona a [[Chamada estática vs injeção CDI]] e pode ser configurado com [[Quarkus]].

Interfaces não obrigam o uso de injeção de dependência. Elas também podem ser usadas diretamente em bibliotecas, algoritmos e componentes locais.

## Interfaces e Strategy Pattern

O [[Strategy Pattern]] costuma usar uma interface para representar algoritmos intercambiáveis:

```java
public interface RegraDesconto {
    BigDecimal aplicar(BigDecimal valor);
}
```

Cada estratégia implementa a mesma operação. O Context recebe uma `RegraDesconto` e não precisa conhecer a fórmula concreta.

Uma [[Factory Pattern]] pode escolher e criar a implementação correta. A interface define o contrato; a factory decide a criação; o Strategy representa o comportamento variável.

## Interfaces em um backend

Em um [[Backend]], interfaces são úteis para separar regras de negócio de detalhes como:

- gateway de pagamento;
- envio de e-mail;
- armazenamento de arquivos;
- acesso a banco;
- publicação de mensagens;
- consulta a serviços externos;
- relógio e geração de identificadores.

Não crie uma interface para cada classe automaticamente. Uma interface agrega valor quando existe uma variação real, uma fronteira externa, uma necessidade de teste ou um contrato que precisa ser preservado.

## Testes com interfaces

Uma implementação falsa pode ser usada para testar uma classe sem chamar um serviço real:

```java
GatewayPagamento gatewayFake = valor ->
        ResultadoPagamento.aprovado();

var service = new PagamentoService(gatewayFake);
var resultado = service.pagar(new BigDecimal("20.00"));
```

O teste pode verificar o comportamento do serviço sem depender de rede, banco ou provedor de pagamento. Ainda é necessário testar a implementação real em testes de integração.

Boas verificações incluem:

- se o contrato é respeitado por cada implementação;
- se erros são tratados corretamente;
- se valores inválidos são rejeitados;
- se chamadas externas recebem os parâmetros corretos;
- se a implementação falsa não esconde um comportamento importante.

O assunto se conecta a [[Testes]] e [[Polimorfismo]].

## Interfaces e SOLID

Interfaces ajudam especialmente em dois princípios do [[SOLID]]:

- **Dependency Inversion Principle:** o código de alto nível depende de abstrações;
- **Interface Segregation Principle:** clientes não devem ser obrigados a depender de métodos que não usam.

Uma interface enorme não é uma boa abstração. Em vez de criar um `ServicoCompleto` com dezenas de métodos, considere contratos menores e específicos:

```java
public interface LeitorPedido {
    Pedido buscar(long id);
}

public interface GravadorPedido {
    void salvar(Pedido pedido);
}
```

Uma classe pode implementar os dois contratos quando fizer sentido, mas cada consumidor depende somente da capacidade necessária.

## Interface e segurança

Interfaces ajudam na organização, mas não são um mecanismo de segurança. A implementação concreta ainda precisa:

- validar entradas;
- verificar autenticação e autorização;
- proteger dados sensíveis;
- controlar permissões no servidor;
- tratar falhas de integrações;
- evitar expor detalhes internos.

Não escolha a implementação de uma interface usando diretamente um nome arbitrário recebido pela [[Requisição]]. Use uma lista permitida e valide a configuração. As regras continuam fazendo parte de [[Segurança]].

## Armadilhas comuns

- criar interfaces vazias ou artificiais apenas para cumprir uma regra;
- criar uma interface para cada classe sem existir variação real;
- colocar métodos demais em um único contrato;
- usar uma interface para esconder dependências globais;
- confundir interface com implementação;
- adicionar métodos e quebrar todas as implementações sem planejar a evolução;
- usar implementações padrão para esconder regras complexas;
- criar uma abstração tão genérica que ninguém sabe qual comportamento esperar.

Uma abstração deve tornar uma mudança provável mais segura ou tornar o código mais fácil de testar. Se apenas acrescenta arquivos e indireção, talvez não seja necessária.

## Boas práticas

- nomeie a interface pelo papel ou capacidade, como `GatewayPagamento`;
- mantenha o contrato pequeno e coeso;
- documente pré-condições, resultado e erros;
- prefira interfaces no limite entre seu código e sistemas externos;
- mantenha as implementações substituíveis;
- use `@Override`, `override` e equivalentes do compilador;
- evite expor detalhes de uma implementação no contrato;
- teste o contrato e as implementações importantes;
- use composição para combinar capacidades;
- avalie classe abstrata quando houver estado e comportamento compartilhados;
- não confunda abstração com segurança ou validação;
- use [[Linting]] e avisos do compilador para encontrar assinaturas incorretas.

## Em uma frase

**Uma interface abstrata é um contrato de capacidades que permite usar implementações diferentes de forma uniforme, sem revelar como cada uma funciona internamente.**

### Veja também

- [[Classe abstrata]]
- [[Polimorfismo]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[Design Pattern]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
- [[Segurança]]

## Fontes

- [Interfaces — Java Language Specification](https://docs.oracle.com/javase/specs/jls/se17/html/jls-9.html)
- [Interface keyword — C# Reference](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/interface)
- [Abstract classes — C++](https://learn.microsoft.com/en-us/cpp/cpp/abstract-classes-cpp?view=msvc-170)
