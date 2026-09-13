# O que significa `public`?

`public` é um modificador de acesso. Ele indica que uma classe, método, propriedade, campo ou outro membro pode ser acessado por código que esteja fora do local onde foi declarado, respeitando as regras específicas da linguagem.

Uma analogia: imagine uma porta de uma casa. Uma porta `public` é uma entrada autorizada para visitantes. Isso não significa que qualquer pessoa possa fazer qualquer coisa dentro da casa; significa que aquela entrada faz parte do acesso oferecido.

`public` define visibilidade, mas não garante que o código seja seguro, correto, imutável ou thread-safe. Essas preocupações precisam ser tratadas separadamente.

## Por que controlar acesso?

Controlar visibilidade ajuda a:

- proteger o estado interno de um objeto;
- definir uma API pública estável;
- reduzir acoplamento;
- impedir alterações indevidas;
- deixar claro o que outras partes podem usar;
- facilitar mudanças internas sem quebrar consumidores.

O modificador se relaciona a [[Classe abstrata]], [[Interface abstrata]], [[Implements]], [[Extends]] e [[Polimorfismo]].

## `public` em Java

Em [[Java]], uma classe e seus membros podem ser públicos:

```java
public final class Usuario {
    private final String nome;

    public Usuario(String nome) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome obrigatório");
        }

        this.nome = nome;
    }

    public String nome() {
        return nome;
    }
}
```

Nesse exemplo:

- `public class Usuario` permite que outros pacotes usem o tipo;
- o construtor `public` permite criar usuários de fora da classe;
- o método `public nome()` oferece uma forma controlada de consultar o nome;
- o campo `private` impede que outras classes alterem o estado diretamente;
- `final` impede que outra classe estenda `Usuario`.

Uma classe pública de nível superior normalmente precisa estar em um arquivo com o mesmo nome, como `Usuario.java`. Uma unidade de compilação pode ter outras classes sem `public`, mas somente uma classe pública principal com aquele nome.

## Níveis de acesso em Java

Java possui quatro níveis comuns:

| Modificador | Acesso aproximado |
|---|---|
| `public` | qualquer código que consiga acessar o tipo e o módulo |
| `protected` | a própria classe, o mesmo pacote e subclasses, com regras específicas |
| sem modificador | o mesmo pacote (*package-private*) |
| `private` | somente a própria classe, com exceções para recursos aninhados |

O acesso de módulos, pacotes e classes envolvidas também pode limitar o que realmente está visível. `public` não ignora todas as regras de encapsulamento do módulo.

## Métodos públicos e contrato

Todo método `public` pode virar parte do contrato da classe:

```java
public BigDecimal calcularTotal(List<Item> itens) {
    // Regra pública da classe.
}
```

Quando outras classes começam a depender desse método, alterar seu nome, parâmetros, retorno ou comportamento pode quebrar consumidores. Por isso, mantenha a superfície pública pequena e estável.

Nem todo método que funciona hoje precisa ser `public`. Se ele só é usado internamente, deixe-o `private` ou com o acesso mais restrito adequado.

## Interfaces e `public`

Métodos de interface em Java são públicos por contrato. Uma implementação não pode reduzir essa visibilidade:

```java
public interface Exportavel {
    byte[] exportar();
}

public final class Relatorio implements Exportavel {
    @Override
    public byte[] exportar() {
        return new byte[0];
    }
}
```

O método da classe precisa continuar `public`. Se fosse `private` ou `protected`, a implementação não cumpriria corretamente o contrato.

Interfaces são detalhadas em [[Interface abstrata]] e a declaração de implementação está em [[Implements]].

## `public` em C#

Em [[C#]], `public` também pode ser usado em tipos e membros:

```csharp
public sealed class Usuario
{
    public Usuario(string nome)
    {
        if (string.IsNullOrWhiteSpace(nome))
        {
            throw new ArgumentException("Nome obrigatório", nameof(nome));
        }

        Nome = nome;
    }

    public string Nome { get; }
}
```

Uma classe pública pode ser usada por outros assemblies, desde que o assembly e seus tipos relacionados estejam acessíveis. Uma classe de nível superior sem modificador costuma ter acesso `internal`, ficando disponível somente dentro do assembly.

Em C#, uma propriedade pública não precisa permitir escrita pública:

```csharp
public decimal Total { get; private set; }
```

Essa forma permite que outras classes consultem o total, mas somente a própria classe altere seu valor. É uma ferramenta importante de encapsulamento.

## `public` em C++

Em [[C++]], `public:` define uma seção da classe cujos membros podem ser acessados por código externo:

```cpp
#include <string>

class Usuario
{
public:
    explicit Usuario(std::string nome)
        : nome_{std::move(nome)}
    {
    }

    const std::string& nome() const
    {
        return nome_;
    }

private:
    std::string nome_;
};
```

Em C++, uma classe `class` começa com membros `private` por padrão, enquanto uma `struct` começa com membros `public`. Mesmo assim, não deixe dados públicos automaticamente: escolha a visibilidade de acordo com o contrato.

O `public` também aparece na herança:

```cpp
class Cachorro : public Animal
{
};
```

Isso indica herança pública e representa uma relação de substituição: um `Cachorro` pode ser usado onde um `Animal` é esperado. Herança é explicada em [[Extends]].

## `public` não significa “tudo liberado”

Um membro público ainda pode:

- validar argumentos;
- exigir autenticação;
- verificar autorização;
- rejeitar estados inválidos;
- lançar exceções;
- ser limitado por módulos, assemblies ou bibliotecas;
- depender de invariantes internas.

Por exemplo, um método público `cancelarPedido()` não deve permitir que qualquer pessoa cancele qualquer pedido. A visibilidade do método é diferente da autorização para executar a operação.

## Campos públicos versus métodos públicos

Evite campos mutáveis públicos:

```java
// Mais difícil de proteger:
public BigDecimal saldo;
```

Qualquer código poderia atribuir um saldo inválido sem passar por validação. Prefira estado privado e métodos com regras:

```java
private BigDecimal saldo = BigDecimal.ZERO;

public void depositar(BigDecimal valor) {
    if (valor.signum() <= 0) {
        throw new IllegalArgumentException("Valor inválido");
    }

    saldo = saldo.add(valor);
}

public BigDecimal consultarSaldo() {
    return saldo;
}
```

O método público controla como a mudança acontece. Isso protege as regras do domínio e se relaciona ao encapsulamento e ao [[SOLID]].

## API pública

Uma **API pública** é o conjunto de classes, métodos, propriedades, endpoints e formatos que outros consumidores podem usar. Nem tudo que é tecnicamente acessível deve ser considerado parte da API que você pretende manter.

Ao tornar algo público, pense:

- quem precisa usar isso?
- o nome e a assinatura são claros?
- o comportamento pode ser mantido no futuro?
- quais erros podem acontecer?
- que dados podem ser expostos?
- é possível testar e documentar esse contrato?

Em uma [[Biblioteca]], uma API pública pode ter consumidores externos. Em um [[Backend]], um método público interno e um endpoint público são coisas diferentes: um endpoint ainda precisa de autenticação, autorização, validação e controles de [[Segurança]].

## `public` e herança

Um membro público da classe-base pode ser acessado pelos consumidores da subclasse, salvo se a linguagem ou a subclasse alterarem essa exposição dentro das regras permitidas.

```java
public class Animal {
    public void mover() {
        System.out.println("Movendo");
    }
}

public final class Peixe extends Animal {
    @Override
    public void mover() {
        System.out.println("Nadando");
    }
}
```

A subclasse pode sobrescrever o método, mas deve preservar o contrato esperado por quem usa `Animal`. Isso se relaciona ao polimorfismo e ao princípio de substituição do [[SOLID]].

## `public` e construtores

Um construtor público permite que o objeto seja criado por código externo:

```java
public Pedido(long id) {
    this.id = id;
}
```

Um construtor privado pode impedir criação direta e forçar uma fábrica ou método controlado:

```java
public final class Configuracao {
    private Configuracao() {
    }

    public static Configuracao padrao() {
        return new Configuracao();
    }
}
```

Esse desenho pode ser útil em casos específicos, mas não transforme todos os construtores em privados sem necessidade. Uma [[Factory Pattern]] pode encapsular criação quando houver uma decisão real.

## Uso em testes

Um método público pode ser testado diretamente, mas a existência de visibilidade pública não deve ser decidida apenas para facilitar o teste. Se uma regra é interna, teste-a por meio do comportamento público da classe ou reveja a divisão de responsabilidades.

Em [[Testes]], interfaces públicas também permitem fornecer fakes e implementações controladas. Já tornar campos internos públicos apenas para o teste normalmente enfraquece o encapsulamento.

## Boas práticas

- use o menor nível de acesso que atende ao contrato;
- mantenha a API pública pequena;
- prefira propriedades somente leitura quando a alteração não deve ser externa;
- evite campos mutáveis públicos;
- valide entradas nos métodos públicos;
- documente efeitos colaterais e erros;
- não exponha entidades internas ou dados sensíveis sem necessidade;
- proteja endpoints com autenticação e autorização;
- não confunda visibilidade com permissão;
- use interfaces para contratos e implementação privada para detalhes;
- preserve o contrato ao sobrescrever métodos;
- considere compatibilidade antes de alterar algo público;
- use [[Linting]] e revisão de código para identificar APIs acidentais;
- escreva [[Testes]] para o comportamento público importante.

## Em uma frase

**`public` torna um tipo ou membro acessível fora do local onde foi declarado, mas não elimina a necessidade de encapsulamento, validação, autorização e segurança.**

### Veja também

- [[Private]]
- [[Protected]]
- [[Final]]
- [[Classe abstrata]]
- [[Interface abstrata]]
- [[Implements]]
- [[Extends]]
- [[Polimorfismo]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Frontend]]
- [[Testes]]
- [[Segurança]]
