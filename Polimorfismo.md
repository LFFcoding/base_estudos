# O que é polimorfismo?

**Polimorfismo** significa “muitas formas”. Na programação, é a capacidade de tratar objetos diferentes por meio de um mesmo contrato, deixando que cada objeto execute a operação da sua própria maneira.

Uma analogia: várias pessoas podem responder ao comando “fale”, mas cada uma usa uma voz diferente. Quem dá o comando não precisa saber se está falando com uma pessoa, um robô ou um gravador; basta que todos entendam a mesma ação.

Em código, o polimorfismo permite escrever:

```text
para cada notificador:
    notificador.enviar(mensagem)
```

O chamador usa `enviar`, mas o resultado pode ser um envio por e-mail, SMS ou outro canal.

## Por que usar?

Polimorfismo ajuda a:

- reduzir `if`s baseados no tipo concreto;
- trocar implementações sem alterar o código consumidor;
- separar contratos de detalhes;
- adicionar comportamentos com menor impacto;
- reutilizar fluxos comuns;
- facilitar testes com implementações falsas.

Ele é muito usado em [[Backend]], aplicações de [[Frontend]], bibliotecas, frameworks e jogos.

## O exemplo principal: uma interface

Uma interface define um contrato comum:

```java
public interface Notificador {
    void enviar(String mensagem);
}
```

Duas classes podem cumprir esse contrato de formas diferentes:

```java
public final class NotificadorEmail implements Notificador {
    @Override
    public void enviar(String mensagem) {
        System.out.println("Enviando e-mail: " + mensagem);
    }
}

public final class NotificadorSms implements Notificador {
    @Override
    public void enviar(String mensagem) {
        System.out.println("Enviando SMS: " + mensagem);
    }
}
```

O código consumidor pode trabalhar com a interface:

```java
List<Notificador> notificadores = List.of(
        new NotificadorEmail(),
        new NotificadorSms());

for (Notificador notificador : notificadores) {
    notificador.enviar("Seu pedido foi atualizado");
}
```

Mesmo que a variável tenha o tipo `Notificador`, cada objeto executa sua própria versão de `enviar`. Essa escolha é chamada de **despacho dinâmico** (*dynamic dispatch*) ou polimorfismo em tempo de execução.

## Herança e sobrescrita

Outra forma comum usa uma classe-base:

```java
public abstract class Forma {
    public abstract double calcularArea();
}

public final class Retangulo extends Forma {
    private final double largura;
    private final double altura;

    public Retangulo(double largura, double altura) {
        this.largura = largura;
        this.altura = altura;
    }

    @Override
    public double calcularArea() {
        return largura * altura;
    }
}
```

Uma função pode receber `Forma` e trabalhar com `Retangulo`, `Circulo` ou outra subclasse sem conhecer cada fórmula. A anotação `@Override` informa que o método substitui um método herdado e permite ao compilador detectar uma assinatura escrita incorretamente.

Use herança quando a subclasse realmente puder ser usada no lugar da classe-base. Se essa relação não for verdadeira, composição ou uma interface podem ser melhores escolhas.

## Sobrescrita versus sobrecarga

Esses termos são parecidos, mas significam coisas diferentes:

### Sobrescrita (*overriding*)

Uma subclasse fornece uma nova implementação para um método herdado com a mesma assinatura:

```java
@Override
public double calcularArea() {
    return largura * altura;
}
```

A implementação usada depende do objeto real em tempo de execução.

### Sobrecarga (*overloading*)

Uma classe possui métodos com o mesmo nome, mas parâmetros diferentes:

```java
public void registrar(String nome) {
    // registra somente o nome
}

public void registrar(String nome, int idade) {
    // registra nome e idade
}
```

Na sobrecarga, o compilador escolhe a versão com base nos argumentos disponíveis. Ela é uma forma de polimorfismo estático, resolvido em tempo de compilação.

## Polimorfismo estático e dinâmico

Há mais de uma forma de falar em polimorfismo:

- **polimorfismo dinâmico:** a implementação é escolhida em tempo de execução, como uma interface apontando para diferentes objetos;
- **polimorfismo estático:** a escolha é feita pelo compilador, como sobrecarga, generics e templates;
- **polimorfismo por parametrização:** uma estrutura trabalha com tipos diferentes, como `List<T>` em Java ou templates em C++.

No dia a dia, quando alguém fala apenas “polimorfismo” em orientação a objetos, geralmente está falando do primeiro caso: usar uma abstração e permitir que o objeto concreto defina o comportamento.

## Polimorfismo em C#

Em [[C#]], interfaces e classes abstratas são usadas de forma parecida:

```csharp
public interface IForma
{
    double CalcularArea();
}

public sealed class Quadrado : IForma
{
    public double Lado { get; }

    public Quadrado(double lado)
    {
        Lado = lado;
    }

    public double CalcularArea()
    {
        return Lado * Lado;
    }
}

IForma forma = new Quadrado(4);
double area = forma.CalcularArea();
```

O código que usa `forma` conhece `IForma`, mas o objeto concreto é `Quadrado`. A propriedade `sealed` impede heranças não planejadas quando a classe não foi feita para ser estendida.

## Polimorfismo em C++

Em [[C++]], o polimorfismo dinâmico normalmente usa funções virtuais:

```cpp
#include <memory>
#include <vector>

class Forma
{
public:
    virtual ~Forma() = default;
    virtual double calcularArea() const = 0;
};

class Quadrado final : public Forma
{
public:
    explicit Quadrado(double lado) : lado_{lado} {}

    double calcularArea() const override
    {
        return lado_ * lado_;
    }

private:
    double lado_;
};

std::vector<std::unique_ptr<Forma>> formas;
formas.push_back(std::make_unique<Quadrado>(4.0));

for (const auto& forma : formas)
{
    double area = forma->calcularArea();
}
```

O destrutor virtual permite destruir corretamente um objeto derivado por meio de uma referência ou ponteiro para a classe-base. `override` pede ao compilador que confirme a sobrescrita, e `final` impede novas derivações de `Quadrado`.

O `std::unique_ptr` ajuda a deixar a posse e o tempo de vida claros. Em C++, polimorfismo e gerenciamento de memória precisam ser pensados juntos.

## Polimorfismo em Java, C# e C++

As três linguagens suportam polimorfismo, mas possuem detalhes próprios:

- Java usa interfaces, classes abstratas e despacho dinâmico; métodos de instância são virtualmente substituíveis, salvo restrições como `final` e `private`;
- C# usa interfaces, classes abstratas e métodos `virtual`, `override` e `sealed`;
- C++ usa funções `virtual`, classes abstratas e controle explícito de destrutores e tempo de vida.

A ideia é a mesma: o consumidor depende do contrato. A sintaxe, o gerenciamento de memória e as regras de extensão são diferentes.

## Polimorfismo e Strategy Pattern

O [[Strategy Pattern]] é um exemplo direto de polimorfismo. O Context depende de uma interface de estratégia, e cada implementação oferece um algoritmo diferente.

```text
Calculadora → Estratégia de desconto
                    ├── desconto normal
                    ├── desconto premium
                    └── desconto promocional
```

O Context pode chamar `calcular` sem conhecer a fórmula usada por cada estratégia. Uma [[Factory Pattern]] pode escolher e criar a implementação adequada.

## Polimorfismo e SOLID

O polimorfismo costuma apoiar princípios do [[SOLID]]:

- **Open/Closed Principle:** novas implementações podem ser adicionadas sem modificar todo o código consumidor;
- **Liskov Substitution Principle:** uma implementação deve poder substituir o contrato sem quebrar expectativas;
- **Dependency Inversion Principle:** módulos de alto nível dependem de abstrações, não de classes concretas.

O princípio de substituição é especialmente importante. Se uma subclasse exige condições que a classe-base não exigia, ou muda o significado de seus métodos, a herança pode estar inadequada.

## Exemplo no backend

Um serviço de pagamento pode depender de uma interface:

```java
public interface GatewayPagamento {
    ResultadoPagamento cobrar(BigDecimal valor);
}
```

Implementações diferentes podem conversar com provedores diferentes. O serviço de negócio usa `GatewayPagamento` sem conhecer detalhes de cada fornecedor.

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

Esse desenho pode ser combinado com injeção de dependência, Factory e testes. A implementação concreta pode ser escolhida pela configuração da aplicação, enquanto a regra principal continua dependente do contrato.

## Polimorfismo e testes

Uma vantagem é substituir a implementação real por uma implementação controlada:

```java
GatewayPagamento gatewayFake = valor ->
        ResultadoPagamento.aprovado();

var service = new PagamentoService(gatewayFake);
```

Assim, o teste não precisa chamar um provedor externo. Ainda é importante testar:

- o contrato comum para cada implementação;
- valores válidos e inválidos;
- erros e timeouts;
- comportamento quando o serviço externo recusa a operação;
- compatibilidade entre o Context e todas as estratégias.

Quando várias classes implementam o mesmo contrato, testes de contrato podem confirmar que todas obedecem às mesmas expectativas.

## Polimorfismo em engines de jogos

Em jogos, polimorfismo pode representar diferentes inimigos, armas, itens ou comportamentos. Um sistema pode chamar `atacar()` em vários tipos de inimigo, e cada um executar sua própria regra.

- na [[Unity Engine]], scripts C# podem implementar interfaces ou herdar de classes-base;
- na [[Unreal Engine]], C++ e Blueprints podem criar variações de Actors e Components;
- no [[Godot Engine]], scripts e nodes podem compartilhar contratos e comportamentos.

O polimorfismo não significa que todas as variações devem formar uma hierarquia profunda. Components e composição frequentemente deixam o sistema mais flexível.

## Composição versus herança

Herança expressa uma relação “é um”. Um `Cachorro` pode ser um `Animal` se respeitar completamente o contrato de `Animal`.

Composição expressa uma relação “tem um”. Um `Personagem` pode ter um `ComportamentoDeMovimento` e trocar esse comportamento sem mudar sua classe.

Prefira composição quando:

- o comportamento precisa ser trocado em tempo de execução;
- uma classe pode combinar vários comportamentos;
- a hierarquia de herança ficaria profunda;
- a relação entre os tipos não é realmente de substituição.

O [[Strategy Pattern]] normalmente usa composição justamente por esse motivo.

## Armadilhas comuns

- criar uma classe-base apenas para compartilhar alguns campos;
- usar `instanceof`, `is` ou `dynamic_cast` em todos os consumidores;
- forçar todas as subclasses a implementar métodos sem sentido;
- criar interfaces enormes;
- esquecer que uma subclasse também precisa respeitar o contrato da base;
- esconder efeitos colaterais atrás de um método com nome genérico;
- criar uma hierarquia tão profunda que ninguém entende qual método será executado;
- usar polimorfismo onde um simples `enum` e uma função seriam mais claros.

Usar verificações de tipo ocasionalmente não torna o código automaticamente errado. A preocupação é quando elas se espalham e fazem o consumidor conhecer todas as implementações.

## Boas práticas

- defina contratos pequenos e claros;
- programe para abstrações quando houver mais de uma implementação ou uma necessidade real de troca;
- use `@Override`, `override` e equivalentes do compilador;
- respeite o contrato da classe-base ou interface;
- prefira composição para comportamentos combináveis;
- mantenha implementações substituíveis e previsíveis;
- trate `null`, erros e efeitos colaterais de forma documentada;
- evite heranças profundas e interfaces gigantes;
- use `final`, `sealed` ou `final` em C++ quando a extensão não for suportada;
- teste cada implementação e o código que depende do contrato;
- use [[Linting]] e avisos do compilador para detectar assinaturas incorretas;
- não use polimorfismo apenas para parecer mais sofisticado.

## Em uma frase

**Polimorfismo permite que objetos diferentes sejam usados por meio de um mesmo contrato, deixando que cada implementação execute o comportamento adequado.**

### Veja também

- [[Design Pattern]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[SOLID]]
- [[Backend]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Testes]]
- [[Linting]]
- [[Unity Engine]]
- [[Unreal Engine]]
- [[Godot Engine]]
