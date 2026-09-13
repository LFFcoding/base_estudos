# O que é uma classe abstrata?

Uma **classe abstrata** (*abstract class*) é uma classe que serve como base para outras classes, mas não pode ser instanciada diretamente. Ela representa uma ideia geral e deixa alguns detalhes para as subclasses.

Uma analogia: uma planta de veículo pode definir que todo veículo precisa acelerar e frear, além de guardar informações comuns, como marca. Mas a forma de acelerar um carro, uma bicicleta e um barco pode ser diferente. A planta é útil como base, mas não é um veículo pronto.

Uma classe abstrata pode ter:

- propriedades ou campos;
- construtores;
- métodos concretos, com implementação;
- métodos abstratos, que apenas definem um contrato;
- regras comuns para todas as subclasses.

Ela se relaciona diretamente a [[Polimorfismo]], [[Design Pattern]] e [[SOLID]].

## Por que usar?

Uma classe abstrata é útil quando várias classes:

- representam uma mesma categoria;
- compartilham estado ou comportamento;
- precisam obedecer a algumas operações obrigatórias;
- possuem uma relação real de “é um” (*is-a*);
- devem reutilizar uma implementação comum.

Ela evita duplicação sem obrigar que todas as classes sejam idênticas.

## Exemplo em Java

```java
public abstract class Forma {
    private final String cor;

    protected Forma(String cor) {
        if (cor == null || cor.isBlank()) {
            throw new IllegalArgumentException("A cor é obrigatória");
        }

        this.cor = cor;
    }

    public String cor() {
        return cor;
    }

    public abstract double calcularArea();

    public String descrever() {
        return "Forma na cor " + cor;
    }
}
```

Nesse exemplo:

- `abstract class` impede `new Forma(...)` diretamente;
- `cor` é um dado comum às formas;
- o construtor `protected` pode ser usado pelas subclasses;
- `calcularArea` é abstrato e não possui implementação;
- `descrever` já possui uma implementação reutilizável.

Uma subclasse concreta precisa implementar o método abstrato:

```java
public final class Retangulo extends Forma {
    private final double largura;
    private final double altura;

    public Retangulo(String cor, double largura, double altura) {
        super(cor);

        if (largura <= 0 || altura <= 0) {
            throw new IllegalArgumentException("Dimensões inválidas");
        }

        this.largura = largura;
        this.altura = altura;
    }

    @Override
    public double calcularArea() {
        return largura * altura;
    }
}
```

Uso polimórfico:

```java
Forma forma = new Retangulo("azul", 4, 3);

double area = forma.calcularArea();
String descricao = forma.descrever();
```

A variável é do tipo `Forma`, mas o objeto concreto é `Retangulo`. A chamada de `calcularArea` usa a implementação da subclasse. Isso é polimorfismo.

## Método abstrato e método concreto

Um método abstrato define uma obrigação:

```java
public abstract double calcularArea();
```

Ele não possui corpo e precisa ser implementado por uma subclasse concreta.

Um método concreto possui uma implementação que pode ser herdada:

```java
public String descrever() {
    return "Forma na cor " + cor;
}
```

Uma subclasse pode usar o método como está ou sobrescrevê-lo quando precisar de um comportamento diferente. Use `@Override` para pedir ao compilador que confirme a sobrescrita.

## Uma classe abstrata pode ter construtor?

Sim. Embora não possa ser criada diretamente, a classe abstrata participa da criação de suas subclasses. O construtor da classe-base é executado por meio de `super`:

```java
public Retangulo(String cor, double largura, double altura) {
    super(cor);
    // Inicialização específica do retângulo.
}
```

O construtor abstrato é um bom lugar para validar invariantes comuns, como um identificador obrigatório ou um valor que não pode ser negativo.

## Classe abstrata em C#

Em [[C#]], a ideia é parecida:

```csharp
public abstract class Forma
{
    protected Forma(string cor)
    {
        if (string.IsNullOrWhiteSpace(cor))
        {
            throw new ArgumentException("A cor é obrigatória", nameof(cor));
        }

        Cor = cor;
    }

    public string Cor { get; }

    public abstract double CalcularArea();

    public virtual string Descrever()
    {
        return $"Forma na cor {Cor}";
    }
}

public sealed class Circulo : Forma
{
    public Circulo(string cor, double raio) : base(cor)
    {
        if (raio <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(raio));
        }

        Raio = raio;
    }

    public double Raio { get; }

    public override double CalcularArea()
    {
        return Math.PI * Raio * Raio;
    }
}
```

No C#:

- `abstract class` impede a criação direta da classe;
- `abstract` exige uma implementação na subclasse;
- `virtual` permite que uma subclasse sobrescreva um método que já possui implementação;
- `override` marca a sobrescrita;
- `sealed` impede novas subclasses de `Circulo`.

## Classe abstrata em C++

Em [[C++]], uma classe que possui pelo menos uma função virtual pura é abstrata:

```cpp
class Forma
{
public:
    virtual ~Forma() = default;
    virtual double calcularArea() const = 0;
};

class Circulo final : public Forma
{
public:
    explicit Circulo(double raio) : raio_{raio} {}

    double calcularArea() const override
    {
        return 3.141592653589793 * raio_ * raio_;
    }

private:
    double raio_;
};
```

O `= 0` torna `calcularArea` uma função virtual pura. A classe `Forma` não pode ser instanciada. O destrutor virtual permite destruir corretamente um objeto derivado por meio da classe-base.

## Classe abstrata versus interface

Uma **interface** normalmente representa um contrato de capacidades. Uma **classe abstrata** pode representar o contrato e também fornecer estado e comportamento compartilhados.

| Classe abstrata | Interface |
|---|---|
| pode possuir campos e propriedades compartilhados | normalmente descreve operações e capacidades |
| pode ter construtor | não costuma controlar a criação do objeto |
| pode fornecer métodos concretos | pode ter implementações padrão, dependendo da linguagem |
| representa uma base comum | pode ser implementada por classes sem uma mesma base |
| costuma formar uma relação “é um” | pode representar capacidades combináveis |

Em Java e C#, uma classe pode herdar de uma classe-base, mas pode implementar várias interfaces. Isso torna interfaces úteis para combinar capacidades sem criar uma hierarquia grande.

Escolha uma classe abstrata quando as subclasses realmente compartilham uma base e invariantes. Escolha uma interface quando o mais importante for o contrato e diferentes classes precisarem oferecer aquela capacidade.

## Classe abstrata versus classe concreta

Uma classe concreta pode ser instanciada diretamente:

```java
var retangulo = new Retangulo("verde", 2, 5);
```

Uma classe abstrata não pode:

```java
// Não compila:
var forma = new Forma("verde");
```

A classe abstrata precisa de uma implementação concreta que complete os métodos obrigatórios.

## Classe abstrata e Template Method

O padrão **Template Method** costuma usar uma classe abstrata. A classe-base define o esqueleto de um processo e deixa algumas etapas para as subclasses:

```java
public abstract class Importador {
    public final void importar() {
        String dados = ler();
        validar(dados);
        salvar(dados);
    }

    protected abstract String ler();

    protected void validar(String dados) {
        if (dados == null || dados.isBlank()) {
            throw new IllegalArgumentException("Dados vazios");
        }
    }

    protected abstract void salvar(String dados);
}
```

Um `ImportadorCsv` e um `ImportadorJson` podem implementar `ler` e `salvar`, enquanto o fluxo principal permanece igual. Não transforme toda classe abstrata em Template Method; ele é apenas uma das formas de usá-la.

## Relação com Strategy e Factory

- [[Strategy Pattern]] normalmente usa composição para trocar um comportamento;
- uma classe abstrata normalmente usa herança para compartilhar uma base;
- [[Factory Pattern]] pode criar uma subclasse concreta e devolvê-la pelo tipo abstrato;
- [[Polimorfismo]] permite chamar métodos da classe-base e executar a implementação concreta.

Se o comportamento precisa mudar frequentemente durante a execução, Strategy pode ser mais flexível que uma hierarquia de classes. Se existe um fluxo fixo com etapas variáveis, uma classe abstrata pode ser adequada.

## Uso no backend

Em um [[Backend]], uma classe abstrata pode representar uma base comum para adaptadores, processadores ou serviços que realmente compartilham regras:

```text
Processador de arquivo
├── Processador CSV
├── Processador JSON
└── Processador XML
```

Use essa estrutura somente quando os processadores tiverem um fluxo e um contrato realmente relacionados. Se cada integração tiver regras muito diferentes, interfaces e composição podem evitar uma classe-base artificial.

## Classes abstratas e testes

Não se testa uma classe abstrata criando-a diretamente. Há duas abordagens comuns:

- testar o comportamento comum por meio de uma subclasse concreta;
- criar uma subclasse de teste mínima que implemente os métodos abstratos.

Também é importante testar cada implementação concreta e verificar o contrato comum. O polimorfismo só é seguro se todas as subclasses respeitarem as expectativas da classe-base.

Isso se relaciona a [[Testes]] e ao princípio de substituição do [[SOLID]].

## Armadilhas comuns

- criar uma classe abstrata apenas para compartilhar alguns campos;
- colocar regras de muitos domínios em uma classe-base;
- usar métodos abstratos que algumas subclasses não conseguem implementar corretamente;
- criar uma hierarquia muito profunda;
- acessar detalhes de subclasses por meio de muitos `instanceof` ou `dynamic_cast`;
- esquecer de proteger invariantes no construtor;
- adicionar métodos vazios para que subclasses “não precisem” implementar tudo;
- usar herança quando composição seria mais simples.

Uma classe-base com métodos vazios pode esconder um contrato mal definido. Se uma operação não faz sentido para todas as subclasses, talvez ela pertença a uma interface ou capacidade separada.

## Boas práticas

- use uma classe abstrata somente quando houver uma relação real de base e subtipos;
- mantenha o contrato pequeno e estável;
- coloque na classe-base apenas comportamento realmente comum;
- valide invariantes comuns no construtor;
- use `@Override`, `override` e `virtual` corretamente;
- use `final` ou `sealed` quando a extensão não for suportada;
- prefira composição quando comportamentos precisarem ser combinados ou trocados;
- teste a classe-base por meio de implementações concretas;
- teste todas as subclasses importantes;
- evite que subclasses dependam de detalhes internos desnecessários;
- documente quais métodos podem ou devem ser sobrescritos;
- mantenha a hierarquia fácil de entender.

## Classes abstratas em engines de jogos

Em engines como [[Unity Engine]], [[Unreal Engine]] e [[Godot Engine]], uma classe abstrata pode definir comportamentos comuns para personagens, armas ou inimigos.

Por exemplo, diferentes inimigos podem compartilhar vida e dano, mas implementar ataques específicos. Porém, cada engine possui seu próprio sistema de componentes e ciclo de vida. Muitas vezes, composição por Components é mais flexível que uma árvore profunda de classes abstratas.

## Em uma frase

**Uma classe abstrata é uma base não instanciável que combina contrato, estado e comportamento comum para ser completada por classes concretas.**

### Veja também

- [[Polimorfismo]]
- [[Design Pattern]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
- [[Unity Engine]]
- [[Unreal Engine]]
- [[Godot Engine]]
