# Princípios SOLID

**SOLID** é um conjunto de cinco princípios para orientar o desenho de código orientado a objetos. Eles ajudam a criar componentes mais fáceis de entender, testar, alterar e reutilizar.

SOLID não é uma lista de regras que deve ser aplicada de forma automática. O objetivo é reduzir dependências desnecessárias e tornar as mudanças mais seguras.

## Uma analogia

Imagine uma cozinha profissional:

- uma pessoa pode cuidar dos pedidos;
- outra pode preparar os ingredientes;
- outra pode cozinhar;
- cada estação tem uma responsabilidade clara;
- uma estação pode ser trocada sem reconstruir a cozinha inteira.

Um sistema bem organizado funciona de modo parecido. Classes e componentes colaboram, mas não precisam conhecer todos os detalhes uns dos outros.

## O que significa SOLID?

- **S — Single Responsibility Principle**: princípio da responsabilidade única;
- **O — Open/Closed Principle**: aberto para extensão e fechado para modificação;
- **L — Liskov Substitution Principle**: princípio da substituição de Liskov;
- **I — Interface Segregation Principle**: princípio da segregação de interfaces;
- **D — Dependency Inversion Principle**: princípio da inversão de dependência.

## S — Responsabilidade única

Uma classe deve ter um motivo principal para mudar. Isso não significa que ela só pode ter um método; significa que suas tarefas devem pertencer ao mesmo assunto.

Uma classe `PedidoService` que calcula preços, grava no banco, envia e-mail e gera PDF possui responsabilidades demais. Uma mudança no formato do e-mail pode exigir alterar a classe de pedidos.

Uma divisão mais clara seria:

```text
PedidoService       -> coordena o caso de uso
CalculadoraPedido   -> calcula o total
PedidoRepository    -> salva e busca dados
EmailSender         -> envia mensagens
RelatorioPedido     -> gera relatório
```

Cada componente pode ser testado com mais facilidade usando [[Testes]].

## O — Aberto para extensão, fechado para modificação

O código deve permitir adicionar comportamentos sem alterar repetidamente uma classe estável.

Em vez de acumular `if` para cada forma de pagamento:

```java
interface CalculadoraDeTaxa {
    BigDecimal calcular(Pedido pedido);
}

final class TaxaPix implements CalculadoraDeTaxa {
    public BigDecimal calcular(Pedido pedido) {
        return BigDecimal.ZERO;
    }
}

final class TaxaCartao implements CalculadoraDeTaxa {
    public BigDecimal calcular(Pedido pedido) {
        return pedido.total().multiply(new BigDecimal("0.02"));
    }
}
```

O serviço pode receber uma implementação de `CalculadoraDeTaxa`. Para adicionar uma nova forma, como boleto, cria-se outra implementação sem reescrever a regra central.

“Fechado para modificação” não significa nunca alterar código. Se a regra estiver errada, ela deve ser corrigida. O princípio evita mudanças espalhadas e desnecessárias quando surge uma nova variação.

## L — Substituição de Liskov

Uma implementação deve poder ocupar o lugar do tipo que ela promete representar sem quebrar as expectativas do código.

Se uma interface diz que um arquivo pode ser lido, uma implementação que sempre lança “operação não suportada” provavelmente não deveria implementar essa interface.

```java
interface LeitorDeArquivo {
    String ler(String caminho);
}

final class LeitorTexto implements LeitorDeArquivo {
    public String ler(String caminho) {
        return "conteúdo";
    }
}
```

Quem usa `LeitorDeArquivo` espera que `ler` funcione para a implementação recebida. A substituição deve preservar contratos, pré-condições, resultados e erros esperados.

Uma herança que apenas reutiliza código, mas quebra essas expectativas, pode ser pior do que uma composição simples.

## I — Segregação de interfaces

Uma classe não deve ser obrigada a depender de métodos que não usa. É melhor ter interfaces pequenas e específicas do que uma interface enorme para todos os tipos de cliente.

Uma interface única poderia ser problemática:

```java
interface ImpressoraCompleta {
    void imprimir();
    void digitalizar();
    void enviarFax();
}
```

Uma impressora simples talvez não digitalize nem envie fax. Interfaces menores representam melhor as capacidades:

```java
interface Imprime {
    void imprimir();
}

interface Digitaliza {
    void digitalizar();
}
```

Assim, cada classe implementa somente os contratos de que precisa.

## D — Inversão de dependência

Módulos de alto nível não devem depender diretamente de detalhes concretos. Ambos devem depender de abstrações.

Em vez de fazer uma regra de negócio criar diretamente um repositório PostgreSQL:

```java
interface UsuarioRepository {
    Optional<Usuario> buscarPorId(long id);
}

final class UsuarioService {
    private final UsuarioRepository repository;

    UsuarioService(UsuarioRepository repository) {
        this.repository = repository;
    }
}
```

O serviço depende de `UsuarioRepository`, não de uma implementação específica. O framework ou o código de configuração fornece a implementação concreta.

Isso facilita trocar uma implementação, usar um repositório falso nos testes ou mudar a forma de persistência. No [[Quarkus]], a injeção de dependências pode ajudar a conectar essas implementações, mas o princípio é maior que qualquer [[Framework]].

## SOLID e a stack do projeto

No [[Backend]] [[Java]] com [[Quarkus]], os princípios podem ajudar a separar:

- recursos HTTP, que recebem [[Requisição|requisições]];
- serviços, que coordenam regras de negócio;
- repositórios, que acessam o [[PostgreSQL]];
- objetos de entrada e saída, que podem ser representados em [[JSON]];
- integrações externas, que devem ficar atrás de interfaces claras.

O [[Maven]] executa compilação e [[Testes]], mas não aplica SOLID automaticamente. A estrutura depende de decisões de design e revisão do código.

## SOLID não significa criar abstrações para tudo

Aplicar SOLID de forma exagerada pode gerar:

- muitas interfaces sem necessidade;
- classes pequenas demais para serem compreendidas;
- camadas que apenas repassam chamadas;
- dificuldade para encontrar onde uma regra realmente está;
- mais código e mais nomes sem benefício claro.

Antes de criar uma abstração, pergunte:

- existe mais de uma implementação ou é provável que exista?
- a mudança realmente ficará isolada?
- a interface descreve uma capacidade útil?
- o código ficará mais fácil de testar e entender?

## Boas práticas

- mantenha classes e métodos com responsabilidades compreensíveis;
- prefira composição a heranças artificiais;
- dependa de contratos pequenos quando houver variações reais;
- injete dependências em vez de criá-las dentro das regras de negócio;
- escreva testes que protejam os comportamentos importantes;
- use nomes que expliquem o papel de cada componente;
- refatore quando a estrutura começar a dificultar mudanças;
- não use SOLID como justificativa para criar complexidade antecipada.

**SOLID é um conjunto de princípios para organizar código orientado a objetos de modo que responsabilidades, extensões e dependências sejam mais fáceis de controlar.**
