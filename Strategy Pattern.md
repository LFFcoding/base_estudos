# O que é Strategy Pattern?

**Strategy Pattern** significa **padrão de estratégia**. É um padrão comportamental que encapsula diferentes algoritmos ou comportamentos atrás de um mesmo contrato, permitindo trocar a estratégia sem alterar o código que a utiliza.

Uma analogia: para chegar ao trabalho, você pode ir de ônibus, carro, bicicleta ou a pé. O destino é o mesmo, mas o caminho usado muda. O objeto principal sabe que precisa de uma forma de transporte; ele não precisa conhecer todos os detalhes de cada opção.

O Strategy Pattern é um dos padrões de [[Design Pattern]]. Ele favorece **composição**: o objeto recebe um comportamento e o utiliza, em vez de possuir uma grande quantidade de condições ou uma hierarquia extensa de herança.

## O problema dos muitos `if`s

Imagine um sistema que calcula frete:

```java
if (tipo.equals("NORMAL")) {
    // calcula frete normal
} else if (tipo.equals("EXPRESSO")) {
    // calcula frete expresso
} else if (tipo.equals("GRATIS")) {
    // calcula frete grátis
}
```

Quando essa decisão se espalha pelo código, cada nova modalidade exige mudanças em vários lugares. As regras ficam misturadas e os testes se tornam mais difíceis.

O Strategy Pattern separa cada cálculo em uma estratégia:

```text
Calculador de frete → contrato de frete → Normal, Expresso ou Grátis
```

## Estrutura do padrão

Os participantes normalmente são:

- **Strategy:** define o contrato comum;
- **Concrete Strategy:** implementa uma variação do algoritmo;
- **Context:** usa uma Strategy sem conhecer seus detalhes;
- **Client:** escolhe ou fornece a estratégia.

O Context não precisa perguntar como a estratégia calcula. Ele somente chama o método definido pelo contrato.

## Exemplo em Java

```java
import java.math.BigDecimal;

public record Pedido(BigDecimal pesoKg) {
}

public interface CalculoFrete {
    BigDecimal calcular(Pedido pedido);
}

public final class FreteGratis implements CalculoFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        return BigDecimal.ZERO;
    }
}

public final class FreteNormal implements CalculoFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        return pedido.pesoKg().multiply(new BigDecimal("1.50"));
    }
}

public final class FreteExpresso implements CalculoFrete {
    @Override
    public BigDecimal calcular(Pedido pedido) {
        return pedido.pesoKg().multiply(new BigDecimal("2.50"));
    }
}

public final class CalculadorFrete {
    private final CalculoFrete estrategia;

    public CalculadorFrete(CalculoFrete estrategia) {
        this.estrategia = estrategia;
    }

    public BigDecimal calcular(Pedido pedido) {
        return estrategia.calcular(pedido);
    }
}
```

Uso:

```java
var pedido = new Pedido(new BigDecimal("4.0"));
var calculador = new CalculadorFrete(new FreteExpresso());

BigDecimal valor = calculador.calcular(pedido);
```

`CalculadorFrete` não conhece a fórmula do frete expresso. Ele depende apenas de `CalculoFrete`. Para trocar a regra, basta fornecer outra estratégia:

```java
var calculador = new CalculadorFrete(new FreteGratis());
```

Esse desenho também respeita a responsabilidade única: cada classe de frete possui uma regra principal para manter.

## Quem escolhe a estratégia?

A escolha pode ser feita pelo código de composição, por configuração ou por uma fábrica:

```java
CalculoFrete estrategia = switch (tipoFrete) {
    case NORMAL -> new FreteNormal();
    case EXPRESSO -> new FreteExpresso();
    case GRATIS -> new FreteGratis();
};

var calculador = new CalculadorFrete(estrategia);
```

Quando essa seleção cresce ou precisa ser reutilizada, uma [[Factory Pattern]] pode concentrar a criação. A factory cria ou escolhe o objeto; o Strategy define o comportamento usado depois da criação.

Evite colocar o mesmo `switch` em todos os consumidores. A decisão deve ficar em um ponto conhecido do sistema.

## Strategy e injeção de dependência

Uma estratégia pode ser entregue por **Dependency Injection**, em vez de ser criada diretamente dentro do Context:

```java
public final class DescontoService {
    private final RegraDesconto estrategia;

    public DescontoService(RegraDesconto estrategia) {
        this.estrategia = estrategia;
    }
}
```

Essa abordagem facilita trocar a regra em diferentes ambientes e usar uma implementação falsa durante os [[Testes]]. Em um [[Backend]], um serviço pode receber estratégias para cálculo de desconto, seleção de transportadora, forma de pagamento ou política de retry.

Frameworks como [[Quarkus]] podem participar da composição e da injeção das implementações. A ideia é relacionada a [[Chamada estática vs injeção CDI]], mas o Strategy Pattern é o desenho do comportamento variável; a injeção é o mecanismo usado para fornecer a implementação.

## Usando funções como estratégia

Quando a regra é pequena e não precisa de estado, uma função pode ser suficiente. Em Java, uma interface funcional pode representar a estratégia:

```java
import java.math.BigDecimal;
import java.util.function.UnaryOperator;

UnaryOperator<BigDecimal> semDesconto = valor -> valor;
UnaryOperator<BigDecimal> dezPorCento =
        valor -> valor.multiply(new BigDecimal("0.90"));

BigDecimal resultado = dezPorCento.apply(new BigDecimal("100.00"));
```

Uma classe nomeada é preferível quando a regra é importante, possui estado, precisa de documentação ou será reutilizada. Uma lambda pode ser melhor para um comportamento curto e local.

## Exemplos de uso

Strategy pode representar:

- formas de calcular frete;
- regras de desconto e impostos;
- métodos de pagamento;
- políticas de autenticação ou autorização;
- algoritmos de [[Sorting]];
- formatos de exportação;
- provedores de armazenamento;
- regras de validação;
- estratégias de retry e backoff;
- diferentes comportamentos de inimigos em jogos.

Na [[Unity Engine]], uma estratégia pode representar comportamentos diferentes de inimigos ou formas de ataque. Na [[Unreal Engine]], a mesma ideia pode ser implementada com C++, Blueprints ou componentes. Em [[Godot Engine]], pode ser feita com GDScript, [[C#]] ou [[C++]].

## Strategy no frontend

No [[Frontend]], uma aplicação pode escolher estratégias para formatar dados, validar formulários, filtrar listas ou adaptar componentes a diferentes plataformas.

Por exemplo, uma tela pode receber uma estratégia de ordenação e mudar o critério sem conhecer o algoritmo:

```text
Lista → estratégia de ordenação → por nome, por data ou por relevância
```

É importante não transformar cada pequena função em uma classe. A abstração vale a pena quando há variação real ou quando o comportamento precisa ser substituído e testado separadamente.

## Strategy e SOLID

O padrão costuma ajudar em alguns princípios do [[SOLID]]:

- **Single Responsibility:** cada estratégia concentra uma regra;
- **Open/Closed:** uma nova estratégia pode ser adicionada sem alterar todas as existentes;
- **Dependency Inversion:** o Context depende de uma abstração, não de uma implementação concreta.

Isso não acontece automaticamente. Se o Context continuar conhecendo todos os detalhes das estratégias ou se a interface ficar cheia de métodos que não pertencem a todas elas, o design ainda precisa ser melhorado.

## Strategy versus outros padrões

### Strategy versus State

- **Strategy:** o cliente ou a composição escolhe o comportamento;
- **State:** o próprio objeto muda de comportamento de acordo com seu estado interno.

Um carrinho pode receber uma estratégia de cálculo de frete. Já um pedido pode mudar internamente de comportamento ao passar de `CRIADO` para `PAGO` e depois para `CANCELADO`, o que se aproxima de State.

### Strategy versus Template Method

- **Strategy:** usa composição e troca de objetos;
- **Template Method:** usa herança e mantém um esqueleto de algoritmo na classe-base.

Prefira Strategy quando a troca em tempo de execução ou a composição deixar o código mais flexível. Use Template Method quando a relação de herança for natural e o fluxo comum precisar ser fixado.

### Strategy versus Factory

- **Factory:** decide ou encapsula como um objeto é criado;
- **Strategy:** encapsula como uma operação ou comportamento é executado.

Uma factory pode criar a Strategy correta. Os padrões podem aparecer juntos, mas resolvem problemas diferentes.

## Testes

Cada estratégia pode ser testada de forma isolada:

```java
@Test
void freteExpressoDeveMultiplicarPesoPeloValorExpress() {
    var estrategia = new FreteExpresso();
    var pedido = new Pedido(new BigDecimal("4.0"));

    assertEquals(
            new BigDecimal("10.00"),
            estrategia.calcular(pedido));
}
```

Também teste o Context com uma estratégia falsa para verificar se ele delega a operação corretamente. Cubra valores zero, valores negativos, limites e entradas inválidas.

O padrão facilita testes porque uma classe pode receber uma estratégia controlada, sem depender de rede, banco ou serviço externo.

## Segurança

Se o tipo da estratégia vier de uma [[Requisição]], use uma lista explícita de opções permitidas:

- não instancie classes arbitrárias a partir de nomes enviados pelo usuário;
- não permita que o cliente escolha uma estratégia que deveria ser exclusiva do servidor;
- valide limites e permissões no [[Backend]];
- mantenha segredos fora das estratégias e da configuração versionada;
- registre falhas sem expor dados sensíveis.

Strategy organiza o comportamento, mas não substitui autenticação, autorização ou outras práticas de [[Segurança]].

## Boas práticas

- modele cada estratégia com uma responsabilidade clara;
- mantenha a interface pequena e estável;
- prefira estratégias sem estado quando isso for possível;
- use objetos imutáveis para configurações;
- escolha a estratégia em um ponto conhecido da aplicação;
- combine com [[Factory Pattern]] quando a criação tiver regras próprias;
- use injeção de dependência quando a troca e os testes se beneficiarem disso;
- use uma lambda somente para comportamentos pequenos e claros;
- evite criar dezenas de classes para diferenças insignificantes;
- documente regras de negócio que não ficam óbvias no nome;
- teste cada estratégia e o Context;
- meça o custo antes de criar estratégias que executam operações pesadas.

## Quando não usar

Talvez Strategy seja exagero quando:

- existe uma única regra que não deve variar;
- o `if` é pequeno, local e mais fácil de entender;
- as alternativas não compartilham um contrato real;
- a interface teria métodos artificiais;
- a quantidade de classes dificultaria mais do que ajudaria.

O objetivo não é eliminar todos os `if`s. É separar variações que realmente precisam evoluir, ser substituídas ou ser testadas de forma independente.

## Em uma frase

**Strategy Pattern encapsula algoritmos ou comportamentos alternativos atrás de um contrato comum, permitindo trocá-los sem modificar o objeto que os utiliza.**

### Veja também

- [[Design Pattern]]
- [[Factory Pattern]]
- [[SOLID]]
- [[Backend]]
- [[Frontend]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Quarkus]]
- [[Testes]]
- [[Segurança]]
