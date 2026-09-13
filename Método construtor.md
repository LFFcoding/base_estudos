# O que é um método construtor?

O termo mais preciso é apenas **construtor** (*constructor*). Ele é uma parte especial de uma classe usada para preparar um novo objeto no momento em que ele é criado.

Uma analogia: antes de entregar um carro novo, a fábrica instala peças obrigatórias, verifica o chassi e deixa o veículo em um estado inicial válido. O construtor faz uma preparação parecida para um objeto.

Um construtor normalmente:

- tem o mesmo nome da classe, em Java e C++;
- não possui tipo de retorno;
- é chamado durante a criação do objeto;
- inicializa campos e propriedades;
- valida valores obrigatórios;
- garante invariantes iniciais;
- pode receber dependências ou configurações.

Apesar de ser chamado informalmente de “método construtor”, ele não é um método comum: não é herdado nem sobrescrito como um método polimórfico.

## Exemplo em Java

```java
public final class Pedido {
    private final long id;
    private final String cliente;

    public Pedido(long id, String cliente) {
        if (id <= 0) {
            throw new IllegalArgumentException("Id inválido");
        }

        if (cliente == null || cliente.isBlank()) {
            throw new IllegalArgumentException("Cliente obrigatório");
        }

        this.id = id;
        this.cliente = cliente;
    }

    public long id() {
        return id;
    }
}
```

Uso:

```java
var pedido = new Pedido(10, "Ana");
```

O construtor valida os dados antes de deixar o objeto disponível. Depois de criado, `id` e `cliente` não podem ser trocados porque foram declarados `final`.

## Construtor não tem retorno

Um construtor não declara `void` nem outro tipo:

```java
public Pedido(long id) {
    this.id = id;
}
```

Isto é um construtor.

```java
public void Pedido(long id) {
    // Isto seria um método chamado Pedido, não um construtor.
}
```

O segundo exemplo possui `void`, então é um método comum. Ele não substitui a inicialização do objeto.

## Construtores sobrecarregados

Uma classe pode ter vários construtores com parâmetros diferentes:

```java
public final class Usuario {
    private final String nome;
    private final String email;

    public Usuario(String nome) {
        this(nome, null);
    }

    public Usuario(String nome, String email) {
        if (nome == null || nome.isBlank()) {
            throw new IllegalArgumentException("Nome obrigatório");
        }

        this.nome = nome;
        this.email = email;
    }
}
```

`this(...)` chama outro construtor da mesma classe. Ele deve ser a primeira instrução do construtor. Isso evita duplicar validações e regras de inicialização.

Não crie dezenas de construtores com combinações confusas. Quando existem muitos parâmetros opcionais, considere um Builder ou uma factory, como explicado em [[Factory Pattern]].

## Construtor padrão

Em Java, se a classe não declara nenhum construtor, o compilador pode fornecer um construtor sem argumentos:

```java
public class Configuracao {
    // O compilador fornece Configuracao() neste caso.
}
```

Assim que você declara um construtor próprio, o construtor sem argumentos não é criado automaticamente:

```java
public class Configuracao {
    public Configuracao(String ambiente) {
    }
}

// new Configuracao(); // Não compila.
```

Se o uso sem argumentos for necessário, declare-o explicitamente e defina um estado inicial válido.

## `this` e `super`

`this` representa a instância atual e pode ser usado para diferenciar campos de parâmetros:

```java
public Pedido(long id) {
    this.id = id;
}
```

`super` chama um construtor da classe-base:

```java
public abstract class Documento {
    protected final String codigo;

    protected Documento(String codigo) {
        this.codigo = codigo;
    }
}

public final class NotaFiscal extends Documento {
    public NotaFiscal(String codigo) {
        super(codigo);
    }
}
```

O construtor da classe-base prepara a parte herdada do objeto. O construtor da subclasse prepara a parte específica. Essa relação usa [[Extends]] e é comum em [[Classe abstrata]].

Se o construtor da classe-base não for público ou protegido de forma compatível, a subclasse não conseguirá chamá-lo. Um construtor privado normalmente impede herança externa.

## Construtores e herança

Construtores não são herdados e não são sobrescritos:

```text
Classe-base:  construtor da base
       ↓ super(...)
Subclasse:    construtor da subclasse
```

Uma subclasse pode oferecer construtores diferentes dos da base, mas precisa inicializar corretamente a parte herdada. Em Java e C#, o construtor da base é executado antes do corpo da subclasse. Em C++, a lista de inicialização constrói as bases e os membros antes do corpo do construtor.

Não confunda construtor com polimorfismo. O método chamado depois da criação pode ser polimórfico; o construtor escolhido é determinado pela forma de criação e pelos argumentos.

## Construtores em C#

Em [[C#]], o construtor tem o mesmo nome da classe e não possui retorno:

```csharp
public sealed class Pedido
{
    public long Id { get; }
    public string Cliente { get; }

    public Pedido(long id, string cliente)
    {
        if (id <= 0)
        {
            throw new ArgumentOutOfRangeException(nameof(id));
        }

        if (string.IsNullOrWhiteSpace(cliente))
        {
            throw new ArgumentException(
                "Cliente obrigatório", nameof(cliente));
        }

        Id = id;
        Cliente = cliente;
    }
}
```

Um construtor da classe-base é chamado com `base`:

```csharp
public sealed class NotaFiscal : Documento
{
    public NotaFiscal(string codigo) : base(codigo)
    {
    }
}
```

C# também possui construtor estático, usado para inicializar o tipo uma vez. Ele não recebe parâmetros e não deve executar trabalho externo pesado sem uma razão clara.

## Construtores em C++

Em [[C++]], inicialize membros na lista de inicialização:

```cpp
#include <stdexcept>
#include <string>
#include <utility>

class Pedido
{
public:
    Pedido(long id, std::string cliente)
        : id_{id}, cliente_{std::move(cliente)}
    {
        if (id_ <= 0 || cliente_.empty())
        {
            throw std::invalid_argument{"Dados inválidos"};
        }
    }

private:
    long id_;
    std::string cliente_;
};
```

A lista de inicialização constrói os membros diretamente. Ela é preferível a criar os membros com um valor qualquer e atribuir outro valor dentro do corpo.

Em C++, também existem construtor de cópia e construtor de movimento. Se a classe administra recursos, como memória, arquivos ou sockets, pense em RAII e nas regras de cópia antes de permitir cópias automáticas.

## Construtor de cópia e movimento em C++

De forma simplificada:

- **construtor de cópia:** cria um objeto copiando outro;
- **construtor de movimento:** transfere recursos de um objeto temporário ou que não será mais usado;
- **destrutor:** libera recursos quando o objeto termina seu tempo de vida.

Tipos da biblioteca padrão, como `std::string` e `std::vector`, já possuem comportamentos bem definidos. Ao criar uma classe que possui recursos próprios, siga a regra dos três, cinco ou zero conforme o design, ou encapsule o recurso em um tipo que já cuide de seu ciclo de vida.

## Construtor de classe abstrata

Uma [[Classe abstrata]] pode ter construtor, mesmo que não possa ser instanciada diretamente:

```java
public abstract class Relatorio {
    protected final String titulo;

    protected Relatorio(String titulo) {
        if (titulo == null || titulo.isBlank()) {
            throw new IllegalArgumentException("Título obrigatório");
        }

        this.titulo = titulo;
    }
}
```

As subclasses chamam esse construtor com `super`. Ele é um bom lugar para validar estado compartilhado entre todas as implementações.

## Construtor privado

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

Esse padrão pode ser usado em:

- classes de utilitários que não precisam de instâncias;
- factories com criação controlada;
- objetos que oferecem métodos nomeados de criação;
- implementações específicas de Singleton, embora Singleton exija avaliação cuidadosa.

Não use construtor privado só para impedir usos que poderiam ser válidos. Ele pode dificultar injeção de dependência, serialização e testes.

## Factory versus construtor

Use um construtor quando:

- existe uma criação direta e clara;
- os parâmetros necessários são poucos;
- o tipo concreto já é conhecido;
- a validação inicial pertence naturalmente ao objeto.

Considere uma [[Factory Pattern]] quando:

- é preciso escolher entre várias classes concretas;
- há nomes de criação mais claros, como `of`, `from` ou `paraClientePremium`;
- a criação envolve etapas ou dependências;
- a classe concreta deve ficar escondida;
- a criação pode retornar uma implementação de uma interface.

Um construtor não precisa carregar toda a lógica de montagem do sistema. Ele deve deixar o objeto válido; uma factory pode coordenar a escolha e a construção.

## Injeção de dependência pelo construtor

Receber dependências no construtor torna o objeto explícito e evita que ele seja criado em um estado incompleto:

```java
public final class PedidoService {
    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
```

Esse estilo é conhecido como **constructor injection** e se relaciona a [[Chamada estática vs injeção CDI]]. Ele facilita testes e deixa claro do que a classe precisa para funcionar.

Valide dependências obrigatórias no construtor:

```java
this.repository = Objects.requireNonNull(repository);
```

Não crie repositórios, clientes HTTP ou conexões de banco diretamente dentro do construtor de um serviço se eles deveriam ser configurados ou substituídos. Em um [[Backend]], deixe a composição da aplicação ou um [[Framework]] cuidar dessas dependências.

## O que evitar no construtor

Evite construtores que:

- fazem chamadas de rede;
- abrem transações;
- gravam no banco sem uma intenção explícita;
- iniciam threads;
- registram listeners que nunca são removidos;
- executam operações demoradas;
- usam métodos sobrescrevíveis antes da subclasse estar pronta;
- expõem `this` para outro objeto durante a construção;
- aceitam dados inválidos e deixam o objeto quebrado.

Construtores devem preparar o estado. Operações demoradas ou efeitos externos geralmente pertencem a métodos explícitos, factories, serviços de inicialização ou etapas controladas do ciclo de vida.

## Construtores e ORM

Ferramentas de [[ORM]] podem exigir um construtor sem argumentos, às vezes com visibilidade protegida ou privada, para reconstruir objetos. Isso é uma exigência técnica do framework, não uma razão para deixar todos os campos públicos.

Mantenha as regras de domínio protegidas e entenda como o ORM cria, hidrata e salva a entidade. Um construtor auxiliar para infraestrutura deve ser separado conceitualmente do caminho usado pela aplicação.

## Testes de construtores

Teste se o construtor:

- aceita os valores válidos;
- rejeita valores nulos, vazios ou fora dos limites;
- inicializa corretamente os campos;
- exige dependências obrigatórias;
- preserva invariantes;
- não executa efeitos externos inesperados.

Exemplo:

```java
@Test
void deveRejeitarIdInvalido() {
    assertThrows(
            IllegalArgumentException.class,
            () -> new Pedido(0, "Ana"));
}
```

Os testes devem verificar o contrato observável do objeto, não detalhes privados irrelevantes. O assunto se conecta a [[Testes]] e [[SOLID]].

## Segurança

Um construtor não substitui autenticação ou autorização. Se recebe dados de uma [[Requisição]], ainda precisa validar formato, tamanho, limites e permissões no [[Backend]].

Boas práticas:

- não coloque senhas ou tokens em valores padrão;
- não registre dados sensíveis no construtor;
- não confie em IDs ou permissões enviados pelo cliente;
- não faça chamadas externas sem timeout e tratamento de falha;
- não deixe o objeto em estado parcialmente válido;
- siga as regras de [[Segurança]].

## Boas práticas resumidas

- deixe o objeto válido ao terminar o construtor;
- valide argumentos obrigatórios cedo;
- use `this`, `super`, `base` e listas de inicialização corretamente;
- prefira campos finais ou somente leitura quando apropriado;
- evite construtores com muitos parâmetros;
- use Builder ou Factory quando a criação ficar complexa;
- injete dependências pelo construtor;
- não execute trabalho externo pesado automaticamente;
- evite chamar métodos sobrescrevíveis durante a construção;
- documente construtores privados e requisitos de criação;
- escreva [[Testes]] para valores válidos e inválidos;
- considere regras do [[ORM]] ou [[Framework]] usado;
- não confunda inicialização com autenticação ou autorização.

## Em uma frase

**Um construtor é o mecanismo especial que inicializa um objeto e deve garantir que ele comece sua vida em um estado válido.**

### Veja também

- [[Classe abstrata]]
- [[Public]]
- [[Private]]
- [[Protected]]
- [[Final]]
- [[Static]]
- [[Extends]]
- [[Implements]]
- [[Polimorfismo]]
- [[Factory Pattern]]
- [[Chamada estática vs injeção CDI]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[ORM]]
- [[Testes]]
- [[Segurança]]
