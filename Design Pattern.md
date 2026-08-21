# O que é Design Pattern?

**Design pattern** significa **padrão de projeto**. É uma solução conhecida para um problema que aparece com frequência no desenvolvimento de software.

Um padrão não é um trecho de código pronto para copiar. Ele é mais parecido com uma receita ou um molde: explica quais partes participam da solução, como elas se relacionam e quais são os benefícios e custos daquela escolha.

Por exemplo, em vez de cada equipe inventar uma maneira diferente de trocar o comportamento de um objeto, ela pode usar o padrão **Strategy**. A ideia é separar cada comportamento em uma estratégia substituível.

## Por que usar padrões?

Padrões ajudam a:

- organizar responsabilidades;
- reduzir soluções improvisadas;
- facilitar a comunicação entre pessoas desenvolvedoras;
- tornar mudanças futuras mais previsíveis;
- reaproveitar ideias que já foram testadas em muitos projetos.

Quando alguém diz “vamos usar uma Factory” ou “essa parte pode ser uma Strategy”, está usando um vocabulário curto para descrever uma estrutura de solução.

## Um padrão não resolve tudo

Usar um padrão sem ter o problema correspondente pode deixar o código mais complicado. O padrão deve tornar o sistema mais fácil de entender e mudar, não apenas aumentar a quantidade de classes.

Uma boa pergunta antes de aplicar um padrão é:

> Qual problema real esta estrutura resolve e o que ficará mais simples depois dela?

Também é importante lembrar que os nomes e as fronteiras podem variar. Um código pode usar a ideia de um padrão sem seguir exatamente um exemplo de livro.

## Categorias tradicionais

Os padrões mais conhecidos foram organizados em três grupos no livro *Design Patterns*, associado aos chamados padrões GoF (*Gang of Four*):

### Padrões de criação (*creational*)

Preocupam-se com a criação de objetos.

- **Factory Method:** concentra a decisão sobre qual objeto criar.
- **Abstract Factory:** cria famílias de objetos relacionados.
- **Builder:** monta um objeto complexo passo a passo.
- **Prototype:** cria um objeto a partir de uma cópia de outro.
- **Singleton:** tenta garantir uma única instância de uma classe.

### Padrões estruturais (*structural*)

Preocupam-se com a forma como classes e objetos são combinados.

- **Adapter:** faz duas interfaces incompatíveis conversarem.
- **Decorator:** acrescenta comportamento sem alterar a classe original.
- **Facade:** oferece uma entrada simples para um conjunto complicado de classes.
- **Proxy:** controla o acesso a outro objeto.

### Padrões comportamentais (*behavioral*)

Preocupam-se com a comunicação e a distribuição de responsabilidades.

- **Strategy:** permite trocar um algoritmo ou comportamento.
- **Observer:** avisa interessados quando algo muda.
- **Command:** transforma uma ação em um objeto.
- **State:** altera o comportamento conforme o estado atual.
- **Template Method:** define o esqueleto de um processo e deixa etapas específicas para subclasses.

Essas categorias são uma forma de organizar o estudo. Um sistema real pode combinar padrões de grupos diferentes.

## Exemplo: Strategy

Imagine um sistema que calcula descontos. A regra pode mudar para clientes comuns, clientes premium ou campanhas especiais. Colocar todos os `if`s em um método enorme dificulta a manutenção.

Com Strategy, cada regra implementa a mesma interface:

```java
public interface RegraDesconto {
    BigDecimal aplicar(BigDecimal total);
}

public final class SemDesconto implements RegraDesconto {
    @Override
    public BigDecimal aplicar(BigDecimal total) {
        return total;
    }
}

public final class DescontoPremium implements RegraDesconto {
    @Override
    public BigDecimal aplicar(BigDecimal total) {
        return total.multiply(new BigDecimal("0.90"));
    }
}
```

O serviço recebe a estratégia e não precisa conhecer os detalhes de cada regra:

```java
public final class CalculadoraDePreco {
    private final RegraDesconto regra;

    public CalculadoraDePreco(RegraDesconto regra) {
        this.regra = regra;
    }

    public BigDecimal calcular(BigDecimal total) {
        return regra.aplicar(total);
    }
}
```

Assim, a aplicação pode usar `new CalculadoraDePreco(new DescontoPremium())` ou outra estratégia. O cálculo principal permanece igual, mas a regra pode ser trocada.

Esse exemplo usa [[Java]] e combina com o princípio de responsabilidade única do [[SOLID]]: cada classe tem uma razão principal para mudar.

## Padrões comuns no backend

Em um [[Backend]], alguns padrões aparecem com bastante frequência:

### Dependency Injection

Na **injeção de dependência (*Dependency Injection*)**, uma classe recebe as dependências de que precisa, em vez de criá-las diretamente.

```java
public class PedidoService {
    private final PedidoRepository repository;

    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
```

O serviço não precisa saber como construir o repositório. Um framework pode fazer essa ligação. [[Quarkus]] possui suporte a injeção de dependência, e [[NestJS]] também organiza seus providers dessa forma.

### Repository

O padrão **Repository** concentra o acesso aos dados. O serviço pode pedir `buscarPorId` sem conhecer os detalhes de SQL ou da implementação do armazenamento.

Ele é comum junto de [[ORM]], [[Prisma]], [[Dapper]] e [[PostgreSQL]]. Porém, não é obrigatório criar um repository para toda classe: uma abstração que apenas repete cada método do ORM pode aumentar o código sem trazer benefício.

### DTO

Um **DTO (*Data Transfer Object*)** transporta apenas os dados necessários entre partes da aplicação, como entre um endpoint e o serviço. Isso evita expor diretamente objetos internos ou entidades do [[ORM]].

O DTO pode ser útil para validar entrada, controlar o formato da resposta e evitar que um campo interno seja alterado por engano.

### Adapter

O **Adapter** coloca uma camada entre o sistema e uma biblioteca ou serviço externo. Se o fornecedor mudar o formato da sua API, o adapter pode absorver a mudança sem espalhá-la pelo projeto.

### Outbox

O [[Outbox Pattern]] é um padrão arquitetural usado quando a aplicação precisa salvar uma alteração no banco e publicar uma mensagem. Ele ajuda a evitar que apenas uma dessas operações seja concluída.

## Padrão, algoritmo e arquitetura são coisas diferentes

- **Design pattern:** solução recorrente para organizar objetos e responsabilidades.
- **Algoritmo:** sequência de passos para resolver um cálculo ou problema, como um método de [[Sorting]].
- **Biblioteca:** código reutilizável que a aplicação chama, como uma ferramenta específica.
- **Framework:** estrutura que orienta o funcionamento da aplicação e chama partes do código, como [[Quarkus]] ou [[NestJS]].
- **Arquitetura:** visão mais ampla das partes do sistema e de como elas se comunicam.

Um padrão pode usar uma biblioteca, viver dentro de um framework e participar de uma arquitetura, mas esses conceitos não são sinônimos.

## Padrões e testes

Um padrão bem escolhido pode facilitar os [[Testes]]. Por exemplo, uma classe que recebe sua dependência por injeção pode receber uma implementação falsa durante o teste.

```java
var repositoryFake = new PedidoRepositoryFake();
var service = new PedidoService(repositoryFake);
```

Mas um padrão não garante que o código esteja correto. É necessário testar regras, erros, limites e integração com dependências reais quando for apropriado.

## Boas práticas

1. **Comece pelo problema.** Não escolha um padrão apenas porque ele é famoso.
2. **Prefira a solução mais simples.** Uma função pequena pode ser melhor que várias classes.
3. **Conheça os custos.** Um padrão pode adicionar indireção, memória, classes e dificuldade de navegação.
4. **Use nomes claros.** O nome deve explicar a responsabilidade, mesmo que o padrão usado seja conhecido.
5. **Mantenha as responsabilidades separadas.** Isso se relaciona aos princípios [[SOLID]].
6. **Evite o excesso de abstração.** Abstrações devem proteger contra uma mudança provável ou facilitar um teste real.
7. **Não confunda Singleton com variável global.** Estado global pode criar dependências escondidas e dificultar testes; use-o somente quando a necessidade for clara.
8. **Documente decisões importantes.** Explique qual problema motivou o padrão e quais alternativas foram consideradas.
9. **Revise com o crescimento do sistema.** Um padrão adequado hoje pode deixar de ser adequado depois.

## Exemplo de decisão

Suponha que um serviço precise enviar notificações por e-mail e SMS:

1. crie uma interface comum, como `Notificador`;
2. implemente `NotificadorEmail` e `NotificadorSms`;
3. injete a implementação escolhida no serviço;
4. teste o serviço usando uma implementação falsa;
5. adicione outra estratégia somente quando houver uma necessidade real.

Essa solução usa ideias de Strategy e Dependency Injection sem exigir uma hierarquia complexa.

## Resumo

Design pattern é um modelo de solução para problemas recorrentes de organização do código. Ele ajuda a comunicar decisões e a manter sistemas flexíveis, mas não deve ser aplicado automaticamente. O melhor padrão é aquele que resolve um problema concreto com o menor aumento necessário de complexidade.

### Veja também

- [[SOLID]]
- [[Backend]]
- [[Java]]
- [[Quarkus]]
- [[NestJS]]
- [[ORM]]
- [[Prisma]]
- [[Dapper]]
- [[PostgreSQL]]
- [[Outbox Pattern]]
- [[Sorting]]
- [[Testes]]
- [[Framework]]
- [[Biblioteca]]
