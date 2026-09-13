# O que significa `protected`?

`protected` é um modificador de acesso que permite usar um membro dentro da própria classe e, normalmente, dentro de classes derivadas. Ele fica entre `public` e `private`: não é aberto para todos, mas é mais acessível que um membro privado.

Uma analogia: imagine uma oficina com uma área reservada para funcionários autorizados. O público não entra, mas pessoas que trabalham na oficina, incluindo especialistas que estendem o serviço, podem acessar alguns recursos internos.

`protected` é especialmente usado em herança. Por isso, deve ser tratado como um ponto de extensão: tudo que é protegido pode virar uma dependência das subclasses e ficar mais difícil de alterar depois.

## Comparação rápida

| Modificador | Ideia geral |
|---|---|
| `public` | qualquer consumidor autorizado pela linguagem pode acessar |
| `protected` | a classe e suas subclasses podem acessar, com regras próprias de cada linguagem |
| `private` | somente a classe que declarou acessa diretamente |
| sem modificador | depende da linguagem, como package-private em Java ou private padrão em C++ `class` |

Mais detalhes sobre `public` e `private` estão em [[Public]] e [[Private]].

## `protected` em Java

Em [[Java]], um membro `protected` pode ser acessado:

- pela própria classe;
- por classes do mesmo pacote;
- por subclasses, mesmo em outro pacote, respeitando as regras de acesso por herança.

```java
public abstract class Relatorio {
    protected final String titulo;

    protected Relatorio(String titulo) {
        this.titulo = titulo;
    }

    protected abstract String montarConteudo();

    public final String gerar() {
        return titulo + "\n" + montarConteudo();
    }
}

public final class RelatorioCsv extends Relatorio {
    public RelatorioCsv(String titulo) {
        super(titulo);
    }

    @Override
    protected String montarConteudo() {
        return "dados,separados,por,virgula";
    }
}
```

Nesse exemplo:

- `titulo` pode ser usado pela classe e pelas subclasses;
- o construtor `protected` permite que subclasses inicializem a base;
- `montarConteudo` é um ponto de extensão protegido;
- `gerar` é público e controla o fluxo completo;
- a subclasse fornece apenas a parte variável.

O código externo pode chamar `gerar`, mas não deveria controlar diretamente a montagem interna:

```java
var relatorio = new RelatorioCsv("Vendas");
String resultado = relatorio.gerar();
```

### Regra importante entre pacotes

Uma subclasse de outro pacote pode usar um membro `protected` por meio da herança, mas não deve ser tratada como se tivesse acesso irrestrito a qualquer instância da classe-base. O acesso protegido de Java possui regras específicas para referências e pacotes.

Para código iniciante, a ideia principal é: `protected` cria uma ponte de extensão entre uma classe-base e suas subclasses, mas não transforma o membro em público.

## `protected` em C#

Em [[C#]], `protected` permite acesso à classe que declarou o membro e aos tipos derivados:

```csharp
public abstract class Processador
{
    protected string Nome { get; }

    protected Processador(string nome)
    {
        Nome = nome;
    }

    protected abstract string ProcessarInterno(string entrada);

    public string Executar(string entrada)
    {
        if (string.IsNullOrWhiteSpace(entrada))
        {
            throw new ArgumentException("Entrada obrigatória", nameof(entrada));
        }

        return ProcessarInterno(entrada);
    }
}

public sealed class ProcessadorTexto : Processador
{
    public ProcessadorTexto(string nome) : base(nome)
    {
    }

    protected override string ProcessarInterno(string entrada)
    {
        return entrada.Trim();
    }
}
```

O método público `Executar` mantém o fluxo e a validação. A subclasse implementa somente `ProcessarInterno`, que é `protected` e não faz parte da API normal para consumidores externos.

C# também possui combinações como `protected internal` e `private protected`, que refinam o acesso entre herança e assembly. Use essas formas apenas quando a fronteira realmente precisar dessa precisão; `protected` simples costuma ser mais fácil de entender.

## `protected` em C++

Em [[C++]], a seção `protected:` permite acesso à própria classe e às classes derivadas:

```cpp
#include <string>
#include <utility>

class Processador
{
public:
    explicit Processador(std::string nome)
        : nome_{std::move(nome)}
    {
    }

    virtual ~Processador() = default;

    std::string executar(const std::string& entrada)
    {
        return processarInterno(entrada);
    }

protected:
    const std::string& nome() const
    {
        return nome_;
    }

    virtual std::string processarInterno(
        const std::string& entrada) = 0;

private:
    std::string nome_;
};
```

Uma classe derivada pode implementar `processarInterno` e usar `nome()`, mas código externo não acessa esses membros diretamente. O método público `executar` é a porta de entrada controlada.

Em C++, `protected` também pode aparecer na declaração de herança:

```cpp
class Especializado : protected Processador
{
};
```

Isso altera como os membros públicos e protegidos da base são expostos pela classe derivada. É diferente de simplesmente declarar um campo na seção `protected` e deve ser usado com cuidado.

## `protected` e classes abstratas

Classes abstratas usam frequentemente membros `protected` para oferecer pontos de extensão às subclasses:

```text
classe abstrata
├── fluxo público e estável
├── validações comuns
└── etapa protected para subclasses
```

Essa estrutura aparece no padrão Template Method, em que a classe-base controla o processo e subclasses completam etapas específicas.

Uma classe abstrata bem desenhada não deve expor todos os seus detalhes como `protected`. Se muitas subclasses precisam conhecer campos internos, a base pode estar acoplando demais suas implementações.

O arquivo [[Classe abstrata]] explica como combinar contrato, estado e comportamento comum.

## `protected` versus `private`

```java
public abstract class Conta {
    private BigDecimal saldo;
    protected final void validarValor(BigDecimal valor) {
        if (valor.signum() <= 0) {
            throw new IllegalArgumentException("Valor inválido");
        }
    }
}
```

`saldo` só pode ser acessado diretamente dentro de `Conta`. `validarValor` pode ser reutilizado por subclasses.

Use `private` quando o detalhe deve ficar totalmente sob controle da classe. Use `protected` somente quando subclasses realmente precisarem participar da implementação.

Muitas vezes, um método `protected` é melhor que um campo `protected`, pois o método pode preservar validações e evitar que a subclasse altere o estado arbitrariamente.

## `protected` versus `public`

Um método `public` faz parte do contrato para consumidores externos. Um método `protected` faz parte principalmente do contrato de extensão:

- público: pensado para quem usa o objeto;
- protegido: pensado para quem implementa ou estende o objeto;
- privado: pensado somente para o funcionamento interno.

Se uma operação precisa ser usada tanto por consumidores externos quanto por subclasses, ela pode ser `public`, mas ainda deve validar entradas e preservar suas invariantes.

## `protected` e interfaces

Interfaces normalmente descrevem capacidades públicas. Uma classe que implementa uma interface precisa oferecer os membros esperados pelo contrato, enquanto pode manter auxiliares `protected` e `private` para sua própria implementação:

```java
public interface Exportavel {
    byte[] exportar();
}

public abstract class ExportadorBase implements Exportavel {
    protected final byte[] normalizar(byte[] dados) {
        return dados.clone();
    }
}
```

O método da interface é a operação pública do contrato; `normalizar` é um detalhe disponível para classes derivadas da base.

O contrato de interfaces e a declaração `implements` estão em [[Interface abstrata]] e [[Implements]].

## `protected` e polimorfismo

Uma subclasse pode sobrescrever um método `protected` e fornecer seu próprio comportamento:

```java
public abstract class Relatorio {
    protected abstract String formatar();

    public final String gerar() {
        return formatar();
    }
}
```

O consumidor chama `gerar`, e o método interno correto é escolhido conforme o objeto concreto. Isso combina herança e [[Polimorfismo]].

Marque métodos sobrescritos com `@Override`, `override` ou o equivalente da linguagem. O compilador ajuda a detectar erros de assinatura.

## Uso no backend

Em um [[Backend]], `protected` pode ser útil em uma base de processadores, validadores ou adaptadores que compartilham um fluxo:

```text
ProcessadorBase
├── Processador de pedido
├── Processador de pagamento
└── Processador de cancelamento
```

Antes de usar herança, verifique se as subclasses realmente compartilham um contrato e um ciclo de vida. Se cada integração for independente, interfaces e composição podem ser mais simples.

Uma integração externa não deve receber acesso protegido aos segredos ou aos detalhes de segurança somente porque é uma subclasse. Use objetos de configuração controlados e aplique as regras de [[Segurança]].

## Testes

Teste a API pública primeiro. Métodos `protected` fazem parte da implementação ou da extensão e podem ser testados indiretamente por meio dos métodos públicos.

Uma subclasse de teste pode ser útil para verificar uma classe abstrata:

```java
final class ProcessadorDeTeste extends Processador {
    ProcessadorDeTeste() {
        super("teste");
    }

    @Override
    protected String processarInterno(String entrada) {
        return entrada.toUpperCase();
    }
}
```

Não mude membros protegidos para públicos somente para facilitar um teste. Se a lógica protegida estiver grande demais, extraia uma colaboração menor ou reveja a hierarquia.

Esse cuidado faz parte de [[Testes]] e dos princípios [[SOLID]].

## Armadilhas comuns

- usar `protected` em todos os campos para facilitar subclasses;
- criar subclasses que dependem de muitos detalhes internos;
- esquecer as diferenças de pacote, assembly e módulo entre linguagens;
- expor dados sensíveis pela herança;
- transformar uma classe-base em um ponto de acoplamento para todo o sistema;
- sobrescrever métodos protegidos sem preservar o contrato;
- usar herança quando composição seria suficiente;
- confundir acesso protegido com autorização de usuário.

## Boas práticas

- mantenha campos privados sempre que possível;
- prefira métodos `protected` pequenos a estado protegido mutável;
- documente quais membros protegidos podem ser sobrescritos;
- mantenha a classe-base estável e o contrato de extensão claro;
- valide o estado antes de chamar ou expor pontos protegidos;
- use `@Override`, `override` e `virtual` corretamente;
- prefira composição para comportamentos combináveis;
- evite hierarquias profundas;
- teste o comportamento público e os contratos das subclasses;
- não use `protected` como atalho para contornar encapsulamento;
- não confunda visibilidade com autenticação, autorização ou [[Segurança]].

## Em uma frase

**`protected` permite que uma classe e suas subclasses usem um membro, criando uma fronteira de extensão mais restrita que `public` e mais aberta que `private`.**

### Veja também

- [[Public]]
- [[Private]]
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
- [[Testes]]
- [[Segurança]]
