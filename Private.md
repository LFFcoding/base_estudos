# O que significa `private`?

`private` é um modificador de acesso que restringe um membro ao próprio tipo que o declarou. Ele é usado para esconder detalhes internos e impedir que qualquer código externo altere o estado diretamente.

Uma analogia: uma casa pode ter um painel elétrico privado. Os moradores usam interruptores públicos, mas não devem mexer diretamente nos fios internos. A casa oferece operações seguras, enquanto a parte perigosa fica protegida.

`private` é uma ferramenta de encapsulamento, não um mecanismo completo de segurança. Ele organiza o acesso durante a compilação, mas não substitui autenticação, autorização ou proteção de dados.

## Por que usar `private`?

Membros privados ajudam a:

- proteger invariantes do objeto;
- impedir alterações diretas e inválidas;
- esconder detalhes que podem mudar;
- reduzir o acoplamento entre classes;
- manter uma API pública menor;
- deixar claro o que é implementação interna.

O assunto complementa [[Public]], [[Classe abstrata]], [[Interface abstrata]], [[Implements]], [[Extends]] e [[Polimorfismo]].

## Exemplo em Java

```java
public final class Conta {
    private BigDecimal saldo = BigDecimal.ZERO;

    public void depositar(BigDecimal valor) {
        if (valor == null || valor.signum() <= 0) {
            throw new IllegalArgumentException("Valor inválido");
        }

        saldo = saldo.add(valor);
    }

    public BigDecimal consultarSaldo() {
        return saldo;
    }
}
```

Nesse exemplo:

- `saldo` é `private`, então outras classes não podem atribuir um valor diretamente;
- `depositar` é `public` e controla as regras para aumentar o saldo;
- `consultarSaldo` permite leitura sem entregar a possibilidade de substituir o valor;
- a classe protege o estado por meio de operações válidas.

O código abaixo não deve compilar:

```java
Conta conta = new Conta();
conta.saldo = new BigDecimal("-100.00");
```

O consumidor precisa usar os métodos públicos, e esses métodos podem rejeitar dados inválidos.

## `private` em Java

Em [[Java]], um membro `private` só pode ser acessado diretamente dentro da classe que o declarou. Subclasses não acessam esse membro diretamente, nem classes do mesmo pacote.

```java
public class Documento {
    private final String codigo;

    public Documento(String codigo) {
        this.codigo = codigo;
    }

    public String codigo() {
        return codigo;
    }
}
```

`codigo` não pode ser lido diretamente por outra classe. O método público define como esse valor é exposto.

Java também possui classes aninhadas que podem acessar detalhes privados da classe que as envolve, de acordo com as regras da linguagem. Isso não muda o objetivo principal: restringir o acesso externo e proteger a implementação.

## Visibilidade padrão em Java

Se nenhum modificador for escrito, o membro não se torna `private`. Ele recebe acesso de pacote (*package-private*):

```java
class ConfiguracaoInterna {
    // Visível dentro do mesmo pacote.
}
```

Essa diferença é importante:

- `private`: somente a classe;
- sem modificador: classes do mesmo pacote;
- `protected`: a classe, o pacote e subclasses, conforme as regras de acesso;
- `public`: código que conseguir acessar o tipo e o módulo.

O arquivo [[Public]] explica o modificador de acesso público.

## `private` em C#

Em [[C#]], campos e membros de uma classe são privados por padrão quando nenhum modificador de acesso é informado, embora escrever `private` explicitamente possa melhorar a clareza:

```csharp
public sealed class Pedido
{
    private decimal _total;

    public decimal Total => _total;

    public void Adicionar(decimal valor)
    {
        if (valor <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(valor));
        }

        _total += valor;
    }
}
```

`_total` não pode ser alterado diretamente por outra classe. A propriedade `Total` fornece somente leitura, e `Adicionar` controla a alteração.

Uma propriedade também pode ter setter privado:

```csharp
public decimal Total { get; private set; }
```

Assim, outras classes podem consultar o valor, mas somente a própria classe pode alterá-lo.

## `private` em C++

Em [[C++]], membros declarados depois de `private:` só podem ser acessados pela própria classe:

```cpp
#include <stdexcept>

class Conta
{
public:
    explicit Conta(double saldoInicial)
        : saldo_{saldoInicial}
    {
        if (saldoInicial < 0)
        {
            throw std::invalid_argument{"Saldo inválido"};
        }
    }

    void depositar(double valor)
    {
        if (valor <= 0)
        {
            throw std::invalid_argument{"Valor inválido"};
        }

        saldo_ += valor;
    }

    double saldo() const
    {
        return saldo_;
    }

private:
    double saldo_{};
};
```

Em uma `class` C++, os membros são `private` por padrão. Em uma `struct`, eles são `public` por padrão. Mesmo assim, declare as seções explicitamente quando isso melhorar a leitura.

## `private` e herança

Uma subclasse não acessa diretamente os membros privados da classe-base:

```java
public class Animal {
    private int idade;
}

public final class Cachorro extends Animal {
    public void mostrarIdade() {
        // Não compila: idade é private em Animal.
        // System.out.println(idade);
    }
}
```

Se a classe-base precisa oferecer uma extensão controlada, pode fornecer um método `protected` ou `public`. Não transforme todos os campos em `protected` apenas para facilitar subclasses; isso expõe detalhes internos e aumenta o acoplamento.

Uma opção melhor é oferecer um método protegido que preserve as regras:

```java
public abstract class Animal {
    private int idade;

    protected final int idadeAtual() {
        return idade;
    }
}
```

Ainda assim, o método protegido deve ser pequeno, estável e necessário. O tema `protected` pode ser estudado separadamente.

## Encapsulamento não é apenas esconder campos

Encapsulamento significa proteger regras e oferecer operações coerentes. Trocar um campo público por um getter e um setter públicos nem sempre resolve:

```java
public BigDecimal getSaldo() {
    return saldo;
}

public void setSaldo(BigDecimal saldo) {
    this.saldo = saldo;
}
```

Se qualquer valor pode ser atribuído, inclusive um saldo negativo, o estado continua sem proteção. Prefira operações que expressem a intenção:

```java
public void depositar(BigDecimal valor) {
    // valida e soma
}

public void sacar(BigDecimal valor) {
    // valida e verifica o saldo
}
```

Métodos públicos representam ações permitidas; campos privados guardam detalhes que não devem ser alterados sem regras.

## `private` e construtores

Um construtor privado impede a criação direta por código externo:

```java
public final class Configuracao {
    private Configuracao() {
    }

    public static Configuracao padrao() {
        return new Configuracao();
    }
}
```

Esse desenho pode ser útil para uma classe utilitária, uma criação controlada ou uma [[Factory Pattern]]. Porém, não torne construtores privados automaticamente. Isso pode dificultar testes, injeção de dependência e uso natural da classe.

## `private` e métodos auxiliares

Métodos que só apoiam a implementação interna devem ser privados:

```java
public final class ValidadorPedido {
    public void validar(Pedido pedido) {
        validarItens(pedido);
        validarCliente(pedido);
    }

    private void validarItens(Pedido pedido) {
        // Regra interna.
    }

    private void validarCliente(Pedido pedido) {
        // Regra interna.
    }
}
```

Manter auxiliares privados evita que outras classes dependam deles e permite alterar a implementação sem aumentar a API pública.

## `private` e interfaces

Uma classe que implementa uma interface normalmente mantém detalhes privados:

```java
public interface CalculadoraFrete {
    BigDecimal calcular(Pedido pedido);
}

public final class FreteNormal implements CalculadoraFrete {
    private final BigDecimal valorPorQuilo;

    public FreteNormal(BigDecimal valorPorQuilo) {
        this.valorPorQuilo = valorPorQuilo;
    }

    @Override
    public BigDecimal calcular(Pedido pedido) {
        return pedido.pesoKg().multiply(valorPorQuilo);
    }
}
```

O consumidor depende de `CalculadoraFrete`, enquanto `valorPorQuilo` continua privado. Isso combina interfaces, [[Polimorfismo]] e [[Strategy Pattern]].

## `private` e testes

Não torne um método privado público apenas para testá-lo diretamente. Primeiro teste o comportamento público da classe:

- chame a operação pública;
- verifique o resultado observável;
- confirme que os dados inválidos são rejeitados;
- valide efeitos colaterais importantes.

Se uma regra privada ficou grande e difícil de testar, talvez ela pertença a uma classe menor com uma responsabilidade clara. Nesse caso, extraia uma colaboração com uma interface ou classe própria, em vez de expor detalhes internos.

Isso se relaciona a [[Testes]] e aos princípios de responsabilidade do [[SOLID]].

## `private` não é segurança completa

Um campo privado não impede que um usuário envie dados indevidos para uma API. Também não substitui:

- autenticação;
- autorização;
- criptografia;
- validação de entrada;
- proteção de segredos;
- controle de acesso a dados.

Uma classe pode ter campos privados e ainda devolver dados sensíveis em um endpoint público. Em um [[Backend]], proteja também a entrada de uma [[Requisição]] e a resposta em [[JSON]]. Siga as regras de [[Segurança]].

## Interfaces e implementação explícita

Em C#, uma implementação explícita de interface pode manter o membro acessível somente pela interface:

```csharp
public interface ILeitor
{
    string Ler();
}

public sealed class Documento : ILeitor
{
    string ILeitor.Ler()
    {
        return "Conteúdo";
    }
}
```

Isso não é exatamente um método `private` comum, mas produz um acesso restrito ao contrato da interface. Use quando a implementação não fizer parte da API normal da classe e houver uma razão clara para isso.

## Armadilhas comuns

- expor campos mutáveis porque são mais fáceis de usar;
- adicionar getters e setters públicos para tudo;
- usar `private` para esconder uma classe que deveria ter uma responsabilidade separada;
- tornar construtores privados sem considerar testes e composição;
- fazer subclasses dependerem de muitos detalhes `protected`;
- acreditar que `private` protege dados contra acesso malicioso em qualquer situação;
- testar detalhes privados em vez do comportamento público;
- criar métodos privados enormes que escondem várias responsabilidades.

## Boas práticas

- use `private` como padrão para estado interno;
- ofereça métodos públicos que expressem intenções do domínio;
- valide valores antes de alterar o estado;
- prefira campos imutáveis quando possível;
- evite setters públicos sem regras;
- mantenha métodos auxiliares privados e pequenos;
- exponha somente o contrato necessário;
- preserve invariantes dentro da classe;
- use composição e interfaces quando a variação for real;
- teste comportamento público e extraia classes quando a lógica interna crescer demais;
- não confunda visibilidade com autorização ou [[Segurança]];
- documente decisões públicas e mantenha detalhes privados livres para evoluir.

## Em uma frase

**`private` restringe o acesso aos detalhes internos de um tipo, ajudando a proteger seu estado e suas regras sem substituir mecanismos de segurança.**

### Veja também

- [[Public]]
- [[Protected]]
- [[Final]]
- [[Classe abstrata]]
- [[Interface abstrata]]
- [[Implements]]
- [[Extends]]
- [[Polimorfismo]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
- [[Segurança]]
