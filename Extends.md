# O que significa `extends`?

`extends` é uma palavra-chave usada principalmente em [[Java]] para declarar que uma classe herda de outra classe ou que uma interface estende outra interface.

A classe que estende é chamada de **subclasse** (*subclass* ou *derived class*). A classe usada como base é chamada de **superclasse** (*superclass* ou *base class*).

Uma analogia: uma bicicleta elétrica pode aproveitar características de uma bicicleta comum, como rodas e guidão, mas acrescenta bateria e motor. Ela continua sendo uma bicicleta, porém possui capacidades adicionais.

Herança deve representar uma relação verdadeira de “é um” (*is-a*). Se um objeto apenas usa outro objeto, composição provavelmente será mais adequada.

## Herança de classes em Java

```java
public class Animal {
    protected final String nome;

    protected Animal(String nome) {
        this.nome = nome;
    }

    public void emitirSom() {
        System.out.println("Som genérico");
    }
}

public final class Cachorro extends Animal {
    public Cachorro(String nome) {
        super(nome);
    }

    @Override
    public void emitirSom() {
        System.out.println(nome + ": au au");
    }
}
```

Nesse exemplo:

- `Cachorro extends Animal` declara a herança;
- `Cachorro` recebe o estado e os membros acessíveis de `Animal`;
- `super(nome)` chama o construtor da classe-base;
- `@Override` informa que `Cachorro` substitui um comportamento herdado;
- `final` impede que outra classe herde de `Cachorro`.

O uso pode ser polimórfico:

```java
Animal animal = new Cachorro("Toby");
animal.emitirSom();
```

A variável conhece `Animal`, mas o objeto real é `Cachorro`. A chamada usa a implementação da subclasse. Isso se relaciona a [[Polimorfismo]].

## O que é herdado?

De forma simplificada, uma subclasse pode receber:

- métodos acessíveis;
- campos acessíveis;
- contratos e comportamentos da classe-base;
- métodos `protected` que foram disponibilizados para subclasses.

Alguns elementos possuem regras especiais:

- construtores não são herdados automaticamente;
- membros `private` pertencem à classe-base e não são acessados diretamente pela subclasse;
- métodos `final` não podem ser sobrescritos;
- membros estáticos pertencem à classe em que foram declarados, mesmo que possam ser acessados por uma subclasse;
- a subclasse deve respeitar as regras e invariantes da classe-base.

Uma subclasse não deve alterar o significado de um método herdado de forma surpreendente.

## Classes abstratas com `extends`

`extends` é muito usado para completar uma [[Classe abstrata]]:

```java
public abstract class Documento {
    public abstract String tipo();

    public String identificar() {
        return "Documento: " + tipo();
    }
}

public final class NotaFiscal extends Documento {
    @Override
    public String tipo() {
        return "nota fiscal";
    }
}
```

`Documento` define um contrato e um comportamento comum. `NotaFiscal` completa o método abstrato e pode reutilizar `identificar`.

Se a subclasse não implementar todos os métodos abstratos, ela também precisa ser declarada como `abstract`:

```java
public abstract class DocumentoParcial extends Documento {
    // Continua abstrata porque ainda não implementa tipo().
}
```

## Interfaces com `extends` em Java

Em Java, uma interface também pode estender outras interfaces:

```java
public interface Identificavel {
    String id();
}

public interface Auditavel {
    void registrarAuditoria();
}

public interface EntidadePersistente
        extends Identificavel, Auditavel {
}
```

`EntidadePersistente` combina os contratos das duas interfaces. Uma classe que implementa `EntidadePersistente` precisa cumprir os contratos herdados.

Essa é uma diferença importante:

- uma classe Java pode `extends` apenas uma classe-base;
- uma classe Java pode `implements` várias interfaces;
- uma interface Java pode `extends` várias interfaces.

O tema `implements` é explicado em [[Implements]] e interfaces são detalhadas em [[Interface abstrata]].

## `extends` versus `implements`

| Palavra | Relação | Exemplo |
|---|---|---|
| `extends` | uma classe herda de outra classe | `class Cachorro extends Animal` |
| `extends` | uma interface estende outra interface | `interface Filho extends Pai` |
| `implements` | uma classe cumpre uma interface | `class Pix implements Pagamento` |

`extends` normalmente reaproveita uma implementação ou representa uma especialização. `implements` normalmente declara uma capacidade ou contrato.

Uma classe pode combinar as duas formas:

```java
public final class Relatorio
        extends Documento
        implements Exportavel, Auditavel {
    // Herda Documento e cumpre dois contratos.
}
```

## `extends` em C#

Em [[C#]], a ideia de herança existe, mas a palavra `extends` não é usada. A classe-base e as interfaces são listadas depois de `:`:

```csharp
public abstract class Documento
{
    protected Documento(string id)
    {
        Id = id;
    }

    public string Id { get; }

    public abstract string Tipo();
}

public sealed class NotaFiscal : Documento, IAuditavel
{
    public NotaFiscal(string id) : base(id)
    {
    }

    public override string Tipo()
    {
        return "nota fiscal";
    }

    public void RegistrarAuditoria()
    {
        // Registra a auditoria.
    }
}
```

Em C#:

- uma classe pode herdar de uma classe-base;
- a classe-base aparece primeiro depois de `:`;
- as interfaces aparecem depois, separadas por vírgulas;
- `base(...)` chama o construtor da classe-base;
- `override` substitui um método `virtual` ou `abstract`;
- `sealed` impede novas heranças.

Portanto, `class Filha : Base, IContrato` é o equivalente conceitual a estender uma classe e implementar uma interface.

## `extends` em C++

Em [[C++]], também não existe a palavra-chave `extends`. A herança é declarada depois de `:`:

```cpp
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
```

`public Forma` declara herança pública. A classe `Forma` é uma interface semelhante ou classe abstrata porque possui uma função virtual pura (`= 0`). `override` confirma a sobrescrita e `final` impede novas derivações de `Quadrado`.

C++ permite herança múltipla:

```cpp
class Relatorio : public Documento, public Exportavel
{
};
```

Ela pode ser útil em casos específicos, mas adiciona complexidade, como ambiguidades e o problema do diamante. Use-a somente quando as relações e os contratos forem claros.

## Sobrescrita e `super`, `base` e classe-base

Uma subclasse pode sobrescrever um método para adaptar o comportamento:

```java
public class Animal {
    public void mover() {
        System.out.println("Movendo");
    }
}

public final class Peixe extends Animal {
    @Override
    public void mover() {
        super.mover();
        System.out.println("Nadando");
    }
}
```

`super` chama uma implementação da classe-base em Java. Em C#, usa-se `base`; em C++, pode-se chamar `Base::metodo()`.

Sobrescrever não significa ignorar o contrato original. A nova implementação deve continuar fazendo sentido para quem trabalha com o tipo-base.

## Herança versus composição

Herança expressa:

```text
Quadrado é uma Forma
```

Composição expressa:

```text
Personagem tem um ComportamentoDeMovimento
```

Prefira composição quando:

- o comportamento pode mudar durante a execução;
- uma classe precisa combinar vários comportamentos;
- a hierarquia ficaria profunda;
- a relação “é um” não for totalmente verdadeira;
- você quer evitar depender de detalhes internos da classe-base.

O [[Strategy Pattern]] normalmente usa composição. Uma classe recebe uma estratégia em vez de herdar uma árvore de comportamentos.

Herança não é errada, mas deve representar uma relação estável e substituível. Uma classe derivada precisa poder ser usada onde a classe-base é esperada, como explica o princípio de substituição do [[SOLID]].

## `extends` e padrões de projeto

Alguns padrões usam herança:

- **Template Method:** uma classe abstrata define o esqueleto do algoritmo e subclasses completam etapas;
- **Factory Method:** subclasses podem escolher qual produto criar;
- alguns frameworks fornecem classes-base para extensão.

Outros padrões preferem composição:

- [[Strategy Pattern]] troca algoritmos por objetos;
- [[Facade Pattern]] coordena componentes sem exigir herança;
- [[Factory Pattern]] encapsula a criação e pode devolver uma abstração.

Não escolha herança apenas porque um padrão a utiliza. Comece pelo problema e pelas relações reais do domínio.

## Uso no backend

Em um [[Backend]], herança pode modelar uma base comum para documentos, eventos ou processadores:

```text
Processador
├── Processador de pedido
├── Processador de pagamento
└── Processador de cancelamento
```

Antes de criar essa hierarquia, verifique se as classes realmente compartilham um fluxo e invariantes. Se apenas possuem alguns campos iguais, uma composição ou um objeto de valor pode ser mais claro.

Uma hierarquia de entidades persistentes também deve ser avaliada com cuidado. O mapeamento e as consultas podem ficar mais complexos, e a herança do código não significa automaticamente que a estrutura do banco de dados será ideal.

## Testes de classes derivadas

Teste a classe-base e as subclasses:

- verifique o comportamento comum;
- verifique o comportamento específico de cada derivada;
- confirme que a subclasse respeita as pré-condições e resultados da base;
- teste métodos sobrescritos e chamadas para `super` ou `base`;
- use polimorfismo para testar o contrato comum;
- evite depender de detalhes internos da hierarquia.

Esse cuidado faz parte de [[Testes]]. Uma subclasse que só funciona quando o consumidor conhece seu tipo concreto pode indicar uma violação do contrato.

## Armadilhas comuns

- criar uma hierarquia apenas para reutilizar alguns campos;
- usar `extends` quando a relação “é um” não é verdadeira;
- sobrescrever métodos e mudar suas regras de forma incompatível;
- colocar muitos métodos `protected` na classe-base;
- criar uma classe-base que conhece todas as subclasses;
- usar herança múltipla sem necessidade em C++;
- depender de casts para acessar comportamentos específicos;
- criar subclasses que ignoram ou lançam erro para métodos obrigatórios;
- deixar a classe-base mutável e difícil de proteger;
- esquecer que classes `final` ou `sealed` não podem ser estendidas.

## Boas práticas

- use herança para uma relação real de substituição;
- mantenha a classe-base pequena e estável;
- prefira métodos `protected` apenas quando forem realmente necessários;
- use `@Override`, `override` e `virtual` corretamente;
- valide invariantes no construtor da classe-base;
- documente quais métodos podem ser sobrescritos;
- prefira composição para comportamentos combináveis;
- evite hierarquias profundas;
- respeite [[SOLID]], especialmente o princípio de substituição;
- teste o contrato comum e cada implementação;
- use `final` ou `sealed` quando a extensão não for suportada;
- não confunda reutilização de código com uma relação de domínio.

## Em uma frase

**`extends` declara uma relação de herança ou extensão, permitindo que uma classe ou interface aproveite e especialize o contrato de uma base.**

### Veja também

- [[Implements]]
- [[Interface abstrata]]
- [[Classe abstrata]]
- [[Polimorfismo]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[Facade Pattern]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
