# O que significa `static`?

`static` indica que algo pertence ao tipo ou ao escopo em que foi declarado, e não a cada objeto criado. Em vez de cada instância ter sua própria cópia, existe um membro compartilhado ou uma função que pode ser chamada sem criar um objeto.

Uma analogia: uma sala de aula pode ter um nome individual para cada aluno, mas existe um único quadro compartilhado pela turma. O nome pertence ao aluno; o quadro pertence ao espaço comum.

O significado exato depende da linguagem e do local onde `static` aparece. Em geral, ele indica associação com a classe, com o programa ou com um escopo específico.

## `static` versus instância

Um membro de instância pertence a cada objeto:

```java
public final class Usuario {
    private final String nome;

    public Usuario(String nome) {
        this.nome = nome;
    }
}
```

Cada `Usuario` possui seu próprio `nome`.

Um membro estático pertence à classe:

```java
public final class Usuario {
    private static int quantidadeCriada;

    public Usuario() {
        quantidadeCriada++;
    }

    public static int quantidadeCriada() {
        return quantidadeCriada;
    }
}
```

Uso:

```java
new Usuario();
new Usuario();

int quantidade = Usuario.quantidadeCriada();
```

O contador é compartilhado pelas instâncias. Isso pode ser útil, mas também cria estado global dentro do processo, o que exige cuidado.

## Métodos estáticos

Um método `static` pode ser chamado pelo nome da classe:

```java
public final class Conversor {
    private Conversor() {
    }

    public static int paraInteiro(String valor) {
        return Integer.parseInt(valor);
    }
}

int numero = Conversor.paraInteiro("42");
```

Como o método não está ligado a um objeto específico, ele não pode acessar diretamente campos de instância nem usar `this`:

```java
public class Exemplo {
    private String nome;

    public static void imprimirNome() {
        // Não compila: não existe uma instância específica.
        // System.out.println(nome);
    }
}
```

O método pode acessar outros membros estáticos ou receber um objeto como parâmetro.

## Utilitários e métodos estáticos

Métodos estáticos são adequados quando a operação:

- não depende do estado de uma instância;
- é uma transformação simples e determinística;
- representa uma função utilitária bem definida;
- não precisa de dependências substituíveis;
- não possui efeitos colaterais escondidos.

Exemplos comuns são conversões, validações puras e operações matemáticas. Uma classe utilitária pode ter construtor privado para impedir criação desnecessária.

Não transforme qualquer método em `static` apenas porque ele pode ser chamado sem objeto. A decisão deve refletir o modelo e as dependências da operação.

## `static` e `final`

`static` e `final` resolvem problemas diferentes:

- `static`: o membro pertence à classe ou ao escopo, e não a cada instância;
- `final`: a variável não pode receber outra referência, ou a classe/método não pode ser estendido ou sobrescrito, conforme o caso.

Combinados em Java:

```java
public static final int MAX_TENTATIVAS = 3;
```

Isso representa um valor compartilhado que não pode ser reatribuído. Porém, se o valor for um objeto mutável, `static final` não torna o objeto imutável.

O arquivo [[Final]] explica essa diferença com mais detalhes.

## Constantes

Uma constante normalmente combina um membro compartilhado com uma regra de não alteração:

```java
public static final String STATUS_ATIVO = "ATIVO";
```

Boas constantes:

- têm nomes claros;
- representam valores realmente estáveis;
- não escondem configuração que deveria variar por ambiente;
- não armazenam segredos;
- ficam próximas do domínio em que fazem sentido.

Não coloque toda configuração da aplicação em constantes estáticas. URLs, chaves, limites operacionais e credenciais normalmente precisam vir de configuração segura e validada.

## `static` em Java

Em [[Java]], `static` pode aparecer em:

- campos;
- métodos;
- blocos de inicialização;
- classes aninhadas;
- membros de interfaces, que possuem regras próprias.

Uma classe aninhada `static` não precisa de uma instância da classe externa:

```java
public class Pedido {
    private final long id;

    public Pedido(long id) {
        this.id = id;
    }

    public static class Resumo {
        private final long id;

        public Resumo(long id) {
            this.id = id;
        }
    }
}
```

Uma classe aninhada estática não possui uma referência implícita ao objeto externo. Isso pode evitar retenções de memória e deixar a relação mais clara quando a classe interna não precisa do estado do objeto externo.

### Bloco de inicialização estático

Um bloco estático executa durante a inicialização da classe:

```java
public final class Configuracao {
    private static final Map<String, String> VALORES;

    static {
        VALORES = Map.of(
                "ambiente", "desenvolvimento");
    }
}
```

Use blocos estáticos com moderação. Inicialização complexa pode dificultar a ordem de carregamento, testes e investigação de erros. Um método de construção explícito ou uma configuração injetada costuma ser mais previsível.

## `static` em C#

Em [[C#]], métodos estáticos pertencem ao tipo:

```csharp
public static class Conversor
{
    public static int ParaInteiro(string valor)
    {
        return int.Parse(valor);
    }
}

int numero = Conversor.ParaInteiro("42");
```

Uma classe `static` não pode ser instanciada nem usada como classe-base. Ela deve conter somente membros estáticos.

C# também possui construtor estático, executado uma vez para inicializar o tipo:

```csharp
public static class Ambiente
{
    public static string Nome { get; }

    static Ambiente()
    {
        Nome = "desenvolvimento";
    }
}
```

Use inicialização estática com cuidado quando ela acessar arquivos, rede ou configuração externa. Falhas durante a inicialização podem impedir o uso do tipo inteiro.

## `static` em C++

Em [[C++]], `static` pode significar coisas diferentes conforme o contexto.

### Membro estático de classe

```cpp
class Contador
{
public:
    static int quantidade()
    {
        return quantidade_;
    }

    Contador()
    {
        ++quantidade_;
    }

private:
    inline static int quantidade_{0};
};
```

`quantidade_` é compartilhada pelas instâncias. `inline static` permite definir o membro dentro da classe em versões modernas do C++.

### Função estática de classe

Uma função estática de classe não possui `this` e só acessa diretamente membros estáticos:

```cpp
class Conversor
{
public:
    static int paraInteiro(const std::string& valor)
    {
        return std::stoi(valor);
    }
};
```

### Variável local estática

Uma variável local estática mantém seu valor entre chamadas:

```cpp
int proximoId()
{
    static int id{0};
    return ++id;
}
```

Esse estado persistente pode ser útil, mas deixa a função dependente de memória compartilhada e pode complicar testes e concorrência.

### `static` em escopo de arquivo

Em C++, `static` em uma variável ou função de escopo de arquivo pode limitar sua ligação ao arquivo de tradução. Em código moderno, um namespace sem nome costuma expressar essa intenção com mais clareza. A escolha depende do padrão do projeto e da versão da linguagem.

## Estado estático e concorrência

Um membro estático compartilhado pode ser acessado por várias threads. Isso não o torna automaticamente seguro:

```text
duas threads leem o mesmo contador
        ↓
ambas calculam o próximo valor
        ↓
uma atualização pode sobrescrever a outra
```

Ao compartilhar estado, avalie sincronização, atomicidade, imutabilidade e ciclo de vida. Um contador estático mutável pode causar resultados diferentes dependendo da ordem das operações.

Prefira dados estáticos imutáveis quando possível. Para estado mutável, use mecanismos apropriados da linguagem e teste concorrência quando o sistema exigir.

## `static` e injeção de dependência

Chamadas estáticas escondem dependências:

```java
public final class PedidoService {
    public void salvar(Pedido pedido) {
        BancoGlobal.salvar(pedido);
    }
}
```

O serviço parece não receber dependências, mas depende de `BancoGlobal`. Isso pode dificultar testes, configuração e substituição da implementação.

Uma alternativa é receber um contrato:

```java
public final class PedidoService {
    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }

    public void salvar(Pedido pedido) {
        repository.salvar(pedido);
    }
}
```

O tema se relaciona a [[Chamada estática vs injeção CDI]]. Use métodos estáticos para operações realmente independentes; use injeção quando existir uma dependência externa, variável ou que precise ser substituída em [[Testes]].

## `static` e herança

Membros estáticos são associados ao tipo, não ao objeto concreto. Eles não participam do polimorfismo de instância da mesma forma que métodos virtuais ou sobrescritos.

Em Java e C#, uma subclasse pode declarar um membro estático com o mesmo nome, mas isso normalmente esconde o membro da base em vez de substituí-lo de forma polimórfica. Não use `static` esperando despacho dinâmico.

Se o comportamento precisa variar conforme o objeto real, use um método de instância, uma interface ou uma estratégia. O assunto se conecta a [[Polimorfismo]] e [[Strategy Pattern]].

## `static` e Factory Pattern

Métodos de fábrica estáticos podem oferecer uma forma nomeada de criar objetos:

```java
public final class Pedido {
    public static Pedido vazio() {
        return new Pedido();
    }

    private Pedido() {
    }
}
```

Isso é uma técnica relacionada a [[Factory Pattern]], mas nem todo método estático de criação precisa ser um Factory Pattern completo. Use uma factory quando houver uma decisão, validação, família de produtos ou regra de construção que justifique a separação.

## Uso no backend

Em um [[Backend]], `static` pode ser adequado para:

- funções puras de conversão;
- constantes realmente estáveis;
- tipos utilitários pequenos;
- métodos de criação que não dependem de serviços externos.

Tenha cuidado com:

- caches estáticos sem limite;
- clientes HTTP estáticos mal configurados;
- estado compartilhado entre usuários;
- dados que permanecem depois de uma requisição;
- configurações que deveriam mudar por ambiente;
- testes que dependem da ordem de execução.

Um cache estático pode misturar dados de usuários, crescer sem controle ou permanecer entre testes. Use uma solução apropriada quando o cache for necessário, como [[Redis]], e defina expiração, escopo e regras de invalidação.

## `static` não é singleton automaticamente

Uma classe com métodos ou campos estáticos pode parecer um Singleton, mas os conceitos são diferentes:

- `static` associa membros ao tipo;
- Singleton representa uma instância única controlada;
- uma classe estática não pode ser instanciada;
- um Singleton ainda pode implementar interfaces e ser injetado, embora deva ser usado com cuidado.

Estado estático global costuma esconder dependências e dificultar testes. Não use Singleton apenas para evitar passar uma dependência.

## Testes

Métodos estáticos puros são fáceis de testar:

```java
int resultado = Conversor.paraInteiro("42");
assertEquals(42, resultado);
```

Já estado estático mutável exige limpeza entre testes e pode causar dependência da ordem de execução:

- redefina o estado com segurança;
- evite compartilhar dados entre casos;
- não dependa de testes executados em uma ordem específica;
- use abstrações injetáveis para dependências externas;
- teste concorrência quando houver acesso por várias threads.

O objetivo de [[Testes]] é verificar comportamento, não preservar um desenho estático difícil de substituir.

## Segurança

`static` não protege segredos. Uma constante estática com uma chave de API continua sendo uma chave embutida no programa:

```java
// Não faça isso:
public static final String API_KEY = "segredo";
```

Segredos devem vir de configuração protegida e não devem ser versionados ou enviados ao cliente. Também não mantenha tokens de usuários em campos estáticos, pois o estado pode vazar entre requisições ou usuários.

Siga as regras de [[Segurança]] e valide dados recebidos de uma [[Requisição]].

## Boas práticas

- use `static` quando o comportamento realmente não depender de uma instância;
- prefira métodos estáticos puros e determinísticos;
- mantenha estado estático imutável quando possível;
- não use `static` para esconder dependências externas;
- use `static final` apenas para valores realmente estáveis;
- lembre que referência final ou estática não torna o objeto imutável;
- avalie concorrência e ciclo de vida do estado compartilhado;
- limite e monitore caches estáticos;
- evite tokens, dados de usuário e credenciais em campos estáticos;
- prefira interfaces e injeção para serviços que precisam ser substituídos;
- use `final`, `readonly`, `const` e `sealed` conforme o objetivo e a linguagem;
- teste estado estático com isolamento explícito;
- documente inicialização e efeitos colaterais;
- mantenha a API pública pequena e o estado interno encapsulado.

## Em uma frase

**`static` associa um membro ao tipo ou ao escopo, permitindo compartilhamento sem instância, mas exigindo cuidado com estado global, concorrência, testes e segurança.**

### Veja também

- [[Public]]
- [[Private]]
- [[Protected]]
- [[Final]]
- [[Classe abstrata]]
- [[Interface abstrata]]
- [[Implements]]
- [[Extends]]
- [[Polimorfismo]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[Chamada estática vs injeção CDI]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Redis]]
- [[Testes]]
- [[Segurança]]
