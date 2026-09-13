# O que é Factory Pattern?

**Factory Pattern** significa **padrão de fábrica**. É uma forma de organizar a criação de objetos para que o restante do sistema não precise conhecer todos os detalhes de cada classe concreta.

Uma analogia: em uma lanchonete, o cliente pede um produto no balcão. Ele não precisa entrar na cozinha, escolher ingredientes e montar o lanche. A fábrica recebe o pedido, decide o que criar e entrega um produto pronto.

No código:

- o cliente pede um objeto por meio de uma interface ou método;
- a fábrica decide qual implementação criar;
- o cliente usa o contrato do objeto, sem depender da classe concreta.

Factory é uma família de ideias. Os nomes mais comuns são **Simple Factory**, **Factory Method** e **Abstract Factory**. Todos ajudam a separar a decisão de criação do uso do objeto, mas resolvem problemas diferentes.

Este assunto é uma aplicação específica de [[Design Pattern]].

## O problema que ele resolve

Imagine um sistema que envia notificações. Sem uma fábrica, várias partes da aplicação podem repetir decisões como:

```java
if (tipo.equals("EMAIL")) {
    notificador = new NotificadorEmail();
} else if (tipo.equals("SMS")) {
    notificador = new NotificadorSms();
}
```

Quando essa lógica aparece em muitos lugares, adicionar WhatsApp ou outro canal exige procurar e alterar vários arquivos. A criação fica espalhada e o código passa a conhecer detalhes que não deveria conhecer.

Com uma fábrica, a decisão fica concentrada:

```text
serviço → fábrica → implementação adequada
```

O serviço depende do contrato `Notificador`, e não de todas as classes concretas.

## Simple Factory

**Simple Factory** é uma classe ou método central que cria objetos de tipos diferentes. Não é um padrão formal do catálogo GoF, mas é uma técnica muito comum e útil.

```java
public enum TipoNotificacao {
    EMAIL,
    SMS
}

public interface Notificador {
    void enviar(String mensagem);
}

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

public final class NotificadorFactory {
    private NotificadorFactory() {
    }

    public static Notificador criar(TipoNotificacao tipo) {
        return switch (tipo) {
            case EMAIL -> new NotificadorEmail();
            case SMS -> new NotificadorSms();
        };
    }
}
```

Uso:

```java
Notificador notificador = NotificadorFactory.criar(TipoNotificacao.EMAIL);
notificador.enviar("Seu pedido foi enviado");
```

O código que usa a fábrica conhece `Notificador`, mas não precisa chamar diretamente `new NotificadorEmail()`.

### Vantagens

- centraliza a criação;
- reduz dependência de classes concretas;
- facilita trocar uma implementação;
- deixa o código de uso mais legível;
- oferece um ponto único para validações e configurações de criação.

### Limitações

- a fábrica pode crescer e virar uma classe com muitos `if`s;
- cada novo tipo pode exigir alteração na fábrica;
- existe uma camada extra entre o cliente e o objeto;
- para poucos tipos estáveis, uma fábrica pode ser complexidade desnecessária.

Se a fábrica possui muitas condições que mudam com frequência, pode ser melhor usar registro de implementações, injeção de dependência ou outro desenho mais adequado.

## Factory Method

**Factory Method** é um padrão GoF de criação. Em vez de uma única fábrica decidir tudo, uma classe-base define um método para criar o produto e subclasses escolhem qual produto concreto fornecer.

Exemplo conceitual:

```java
public interface Relatorio {
    void gerar();
}

public abstract class ProcessadorRelatorio {
    public final void processar() {
        Relatorio relatorio = criarRelatorio();
        relatorio.gerar();
    }

    protected abstract Relatorio criarRelatorio();
}

public final class ProcessadorPdf extends ProcessadorRelatorio {
    @Override
    protected Relatorio criarRelatorio() {
        return new RelatorioPdf();
    }
}
```

O método `processar` mantém o fluxo comum, enquanto `criarRelatorio` é o ponto que varia. Uma classe derivada pode criar `RelatorioPdf`; outra pode criar `RelatorioCsv`.

Use Factory Method quando:

- existe um fluxo comum de processamento;
- apenas o tipo de produto criado muda;
- novas variações podem ser representadas por subclasses ou implementações;
- a decisão de criação deve ser deixada para uma extensão do sistema.

Não use herança somente para conseguir uma fábrica. Se composição ou uma função simples resolverem o problema, elas podem ser mais fáceis de manter.

## Abstract Factory

**Abstract Factory** cria famílias de objetos relacionados. A fábrica garante que os objetos escolhidos combinem entre si.

Imagine uma interface que precisa funcionar com tema claro ou escuro:

```java
public interface Botao {
    void desenhar();
}

public interface Janela {
    void desenhar();
}

public interface FabricaInterface {
    Botao criarBotao();
    Janela criarJanela();
}
```

As fábricas `FabricaTemaClaro` e `FabricaTemaEscuro` criam, respectivamente, um botão e uma janela do mesmo tema. O restante do programa recebe uma `FabricaInterface` e não mistura componentes de famílias diferentes.

Abstract Factory é útil quando:

- vários objetos precisam ser compatíveis;
- o sistema trabalha com famílias, temas ou plataformas;
- trocar a família inteira deve ser possível;
- as implementações concretas não devem aparecer no código de uso.

Ele pode criar muitas interfaces e classes. Se houver somente um tipo de produto, Simple Factory ou Factory Method provavelmente serão suficientes.

## Factory e injeção de dependência

Factory e **Dependency Injection** não são a mesma coisa, mas podem trabalhar juntas.

- a factory decide qual objeto criar;
- a injeção de dependência entrega objetos já preparados às classes que precisam deles;
- o framework ou o código de composição conecta as dependências.

Em um [[Backend]], uma fábrica pode escolher o provedor de pagamento conforme a configuração, enquanto o serviço recebe a fábrica por injeção. Em [[Quarkus]], por exemplo, a criação e a injeção podem ser organizadas pelo container CDI.

```java
public final class PedidoService {
    private final GatewayPagamentoFactory factory;

    public PedidoService(GatewayPagamentoFactory factory) {
        this.factory = factory;
    }
}
```

Essa separação facilita trocar implementações e escrever [[Testes]] sem precisar iniciar todos os serviços reais.

## Factory e métodos estáticos

Algumas classes oferecem métodos estáticos como `of`, `from`, `create` ou `valueOf`:

```java
BigDecimal valor = BigDecimal.valueOf(19.90);
```

Esse estilo é uma **static factory method**. Ele pode dar nomes mais claros à criação, devolver uma instância existente, escolher uma subclasse ou validar os dados antes de retornar o objeto.

Um método estático não é automaticamente o Factory Method do padrão GoF. O nome parecido não significa que os padrões sejam iguais; é preciso observar a estrutura e o problema resolvido.

## Factory no C# e no C++

As mesmas ideias podem ser implementadas em [[C#]] e [[C++]]. A sintaxe muda, mas o princípio continua: o consumidor depende de um contrato e a decisão de construção fica isolada.

Em C++, a fábrica também deve respeitar o gerenciamento de memória. Use RAII e smart pointers quando houver posse dinâmica, em vez de retornar ponteiros crus que o chamador pode esquecer de liberar.

Em C#, uma fábrica pode retornar uma interface e usar `switch`, registro de tipos ou integração com o container de dependências. Não use reflexão para instanciar qualquer classe cujo nome venha diretamente de uma entrada do usuário.

## Factory no frontend e em bibliotecas

Factories também aparecem em [[Biblioteca|bibliotecas]] e aplicações de [[Frontend]] para criar componentes, adaptadores ou clientes conforme a configuração. Um [[Framework]] pode esconder fábricas internamente para escolher implementações de plataforma.

Isso não significa que toda função `createSomething` seja um padrão completo. O importante é verificar se a função realmente encapsula uma decisão de criação que seria útil manter separada.

## Factory e configuração

Uma fábrica pode usar configuração, mas a entrada precisa ser validada:

```java
public static Notificador criar(String valor) {
    return switch (valor.toUpperCase(Locale.ROOT)) {
        case "EMAIL" -> new NotificadorEmail();
        case "SMS" -> new NotificadorSms();
        default -> throw new IllegalArgumentException("Tipo não suportado");
    };
}
```

Boas práticas nesse caso:

- use uma lista permitida de opções;
- normalize a entrada antes de comparar;
- rejeite valores desconhecidos;
- não construa classes usando nomes arbitrários recebidos da rede;
- não coloque senhas ou tokens dentro da configuração versionada;
- registre o erro sem revelar dados sensíveis.

Uma fábrica não substitui as regras de [[Segurança]]. Ela apenas organiza a criação.

## Factory e testes

Uma factory pode ser testada para garantir que cada opção retorna a implementação correta:

```java
@Test
void deveCriarNotificadorDeEmail() {
    Notificador resultado = NotificadorFactory.criar(TipoNotificacao.EMAIL);

    assertInstanceOf(NotificadorEmail.class, resultado);
}
```

Também teste entradas inválidas, combinações incompatíveis e falhas de configuração. Além do teste da factory, teste o comportamento do contrato; não dependa somente de verificar o nome da classe criada.

## Factory não é Service Locator

Uma factory explícita recebe parâmetros e devolve um objeto. Já um **Service Locator** costuma esconder dependências em um registro global que qualquer parte do sistema consulta.

O Service Locator pode parecer prático, mas dificulta saber do que uma classe depende e pode tornar os [[Testes]] mais difíceis. Prefira dependências explícitas e use uma fábrica com responsabilidade bem definida.

## Como decidir se vale a pena usar

Use uma Factory quando:

- a criação contém uma decisão ou regra relevante;
- existem várias implementações do mesmo contrato;
- o código de uso não deveria conhecer classes concretas;
- a criação envolve validação, configuração ou recursos externos;
- você prevê novas variações reais.

Talvez não valha a pena quando:

- existe somente uma implementação;
- o construtor já é simples e estável;
- a fábrica apenas repete `return new MinhaClasse()`;
- a abstração não facilita teste nem mudança;
- o sistema passa a ter mais indireção que comportamento.

## Relação com SOLID

Uma Factory pode ajudar o princípio da responsabilidade única do [[SOLID]], pois retira a decisão de criação de uma classe de negócio. Também pode ajudar a depender de abstrações, mas não garante sozinha que o design esteja correto.

Uma fábrica enorme, cheia de condições e dependências, pode estar concentrando responsabilidades demais. Nesse caso, divida o problema, use composição ou reveja a arquitetura.

## Resumo

Factory Pattern organiza a criação de objetos e esconde detalhes das implementações concretas. Simple Factory centraliza a criação, Factory Method permite que extensões escolham o produto e Abstract Factory cria famílias compatíveis.

O padrão é útil quando a criação varia ou possui regras importantes. Se ele apenas adicionar classes e indireção sem resolver um problema real, uma criação direta e simples provavelmente será melhor.

### Veja também

- [[Design Pattern]]
- [[SOLID]]
- [[Backend]]
- [[Frontend]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Quarkus]]
- [[Testes]]
- [[Segurança]]
