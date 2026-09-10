# O que é programação procedural?

**Programação procedural** (*procedural programming*) é um estilo de programação em que o sistema é organizado como uma sequência de instruções e procedimentos.

Um procedimento é um conjunto de passos que executa uma tarefa. O programa chama esses procedimentos na ordem necessária, toma decisões, repete ações e altera valores armazenados na memória.

Uma analogia é uma receita de bolo:

1. separar os ingredientes;
2. misturar alguns ingredientes;
3. colocar a mistura na forma;
4. assar por determinado tempo;
5. retirar e servir.

Na programação procedural, o computador segue instruções parecidas. Cada passo transforma o estado atual até chegar ao resultado.

## As três estruturas principais

Grande parte da programação procedural pode ser organizada com três estruturas:

### Sequência

As instruções são executadas uma depois da outra:

```text
ler preço
calcular desconto
calcular total
mostrar total
```

### Decisão

O programa escolhe um caminho conforme uma condição:

```text
se o pagamento foi aprovado
    confirmar pedido
senão
    informar falha
```

Em Java:

```java
if (pagamentoAprovado) {
    confirmarPedido();
} else {
    informarFalha();
}
```

### Repetição

O programa executa uma ação várias vezes:

```java
for (Produto produto : produtos) {
    calcularPreco(produto);
}
```

O `for` percorre cada produto e chama o procedimento de cálculo.

## Variáveis e estado

Uma variável guarda um valor que pode ser usado ou alterado durante a execução:

```java
int quantidade = 2;
BigDecimal preco = new BigDecimal("10.00");
BigDecimal total = preco.multiply(BigDecimal.valueOf(quantidade));
```

O **estado** é o conjunto de valores que representa a situação atual do programa. Se `quantidade` mudar, o resultado de um cálculo posterior também pode mudar.

Esse estado torna a programação procedural direta e prática, mas pode criar confusão quando muitas partes do sistema alteram as mesmas variáveis.

## Procedimentos e funções

Em muitos idiomas, as palavras **procedimento** (*procedure*) e **função** (*function*) são usadas de maneira parecida. Uma diferença comum é:

- **procedimento:** executa uma ação e pode não devolver um valor;
- **função:** recebe dados, calcula algo e devolve um resultado.

Exemplo de procedimento em Java:

```java
static void exibirMensagem(String mensagem) {
    System.out.println(mensagem);
}
```

Exemplo de função:

```java
static BigDecimal calcularTotal(
        BigDecimal preco,
        int quantidade) {
    return preco.multiply(BigDecimal.valueOf(quantidade));
}
```

O primeiro método produz um efeito, exibindo texto. O segundo devolve um valor que pode ser usado por outro trecho do programa.

Em [[JavaScript]] e [[TypeScript]], a mesma ideia pode ser escrita assim:

```ts
function calcularTotal(preco: number, quantidade: number): number {
  return preco * quantidade;
}

console.log(calcularTotal(10, 2)); // 20
```

## Exemplo completo

O código a seguir calcula o preço final de vários produtos:

```java
static BigDecimal calcularSubtotal(List<Produto> produtos) {
    BigDecimal subtotal = BigDecimal.ZERO;

    for (Produto produto : produtos) {
        subtotal = subtotal.add(
            produto.preco().multiply(
                BigDecimal.valueOf(produto.quantidade())
            )
        );
    }

    return subtotal;
}

static BigDecimal aplicarDesconto(BigDecimal subtotal) {
    if (subtotal.compareTo(new BigDecimal("100.00")) >= 0) {
        return subtotal.multiply(new BigDecimal("0.90"));
    }

    return subtotal;
}
```

O fluxo é procedural:

1. começa com subtotal igual a zero;
2. percorre os produtos;
3. calcula cada valor;
4. acumula os subtotais;
5. verifica uma condição;
6. aplica ou não o desconto;
7. devolve o total.

## Estado local e estado global

Uma variável local existe apenas dentro da função:

```java
static int dobrar(int numero) {
    int resultado = numero * 2;
    return resultado;
}
```

Isso é mais fácil de entender porque outras funções não alteram `resultado` diretamente.

Um estado global pode ser acessado por várias partes:

```java
public static int contadorGlobal = 0;
```

Se vários métodos alterarem `contadorGlobal`, fica mais difícil descobrir quem mudou o valor e em qual ordem. Em aplicações concorrentes, também pode haver problemas de sincronização.

Prefira variáveis locais e passe dados como parâmetros. Use estado compartilhado somente quando ele for realmente necessário e tiver regras claras de acesso.

## Modularização

Um programa procedural pode ser dividido em procedimentos menores:

```text
processarPedido
├── validarPedido
├── calcularTotal
├── reservarEstoque
├── registrarPagamento
└── enviarConfirmacao
```

Cada função deve ter uma responsabilidade compreensível. Isso evita um procedimento gigante que mistura validação, banco, chamadas HTTP, cálculos e mensagens.

Em um [[Backend]], um service pode organizar um fluxo procedural, mas deve separar as responsabilidades em métodos e componentes bem definidos.

## Programação procedural em Java

[[Java]] permite escrever código procedural dentro de métodos, classes e serviços. Mesmo sendo conhecido pela orientação a objetos, Java não impede que uma função siga uma sequência de passos.

```java
public void processarPedido(Pedido pedido) {
    validar(pedido);
    BigDecimal total = calcularTotal(pedido);
    salvar(pedido, total);
    enviarConfirmacao(pedido);
}
```

Esse fluxo pode fazer parte de uma classe gerenciada pelo [[Quarkus]], receber dependências por CDI e usar um [[ORM]] para persistir dados. O fato de o fluxo ser procedural não impede o uso de objetos ou frameworks.

## Programação procedural em JavaScript e TypeScript

[[JavaScript]] e [[TypeScript]] também permitem escrever funções que executam passos sequenciais:

```ts
function prepararResumo(pedido: Pedido): string {
  const subtotal = calcularSubtotal(pedido.itens);
  const desconto = calcularDesconto(subtotal);
  const total = subtotal - desconto;

  return `Total: R$ ${total.toFixed(2)}`;
}
```

Em aplicações com [[Frontend]], como [[React]] e [[Next.js]], funções procedurais podem transformar dados, validar formulários e responder a eventos. No [[Backend]] com [[Node.js]] ou [[NestJS]], elas podem implementar etapas de um serviço.

## Efeitos colaterais

Um **efeito colateral (*side effect*)** acontece quando uma função altera algo fora do seu resultado local, por exemplo:

- grava dados no banco;
- envia uma requisição;
- escreve em um arquivo;
- altera uma variável global;
- envia uma mensagem;
- imprime um log;
- modifica um objeto recebido.

Exemplo com efeito colateral:

```java
void atualizarSaldo(Conta conta, BigDecimal valor) {
    conta.setSaldo(conta.getSaldo().add(valor));
}
```

Uma função pura, por outro lado, depende apenas dos parâmetros e não altera o mundo externo:

```java
BigDecimal somar(BigDecimal a, BigDecimal b) {
    return a.add(b);
}
```

Funções puras são mais fáceis de testar e combinar. Efeitos colaterais são necessários em sistemas reais, mas devem ficar em pontos claros do fluxo.

## Erros e validações

Um programa procedural precisa decidir o que fazer quando algo dá errado:

```java
static BigDecimal calcularMedia(List<BigDecimal> valores) {
    if (valores == null || valores.isEmpty()) {
        throw new IllegalArgumentException("A lista não pode ser vazia");
    }

    BigDecimal soma = BigDecimal.ZERO;
    for (BigDecimal valor : valores) {
        soma = soma.add(valor);
    }

    return soma.divide(BigDecimal.valueOf(valores.size()));
}
```

Valide entradas antes de usá-las, trate exceções no limite correto e não esconda erros com `catch` vazio.

Em uma [[Requisição]], por exemplo, valide o corpo recebido antes de executar a sequência de operações. Uma falha de validação deve gerar uma resposta compreensível, sem expor detalhes internos.

## Procedural e orientação a objetos

Os estilos podem ser combinados:

| Procedural | Orientação a objetos |
|---|---|
| Organiza o código em passos e funções. | Organiza o código em objetos, dados e comportamentos. |
| Dá destaque à sequência da execução. | Dá destaque às responsabilidades e relações entre objetos. |
| Pode ser simples para cálculos e scripts. | Pode ajudar a modelar domínios maiores. |
| Estado costuma ser passado ou alterado por funções. | Estado pode ficar encapsulado no objeto. |

Uma classe Java pode encapsular dados e, dentro de seus métodos, executar um fluxo procedural. Portanto, não é necessário escolher um estilo puro para cada linha do sistema.

## Procedural e programação funcional

Programação funcional dá mais destaque a funções puras, composição e, em muitos casos, imutabilidade. Programação procedural aceita naturalmente mudanças de estado e instruções passo a passo.

Este código é mais procedural porque atualiza `total` em cada repetição:

```java
BigDecimal total = BigDecimal.ZERO;

for (Item item : itens) {
    total = total.add(item.valor());
}
```

Uma solução usando operações de coleção pode esconder parte da sequência, mas continua representando uma transformação de dados:

```java
BigDecimal total = itens.stream()
    .map(Item::valor)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

A escolha deve priorizar clareza. Código mais curto não é automaticamente melhor.

## Procedural e programação declarativa

Na programação procedural, descrevemos **como** executar a tarefa. Na programação declarativa, descrevemos mais o **que** queremos.

Por exemplo, uma consulta ao [[PostgreSQL]] com `SELECT` e `WHERE` é normalmente considerada declarativa: informamos quais dados desejamos e o banco escolhe como executar a busca.

No código procedural, poderíamos percorrer registros e decidir manualmente quais manter. Em geral, é melhor deixar o banco filtrar dados quando isso reduz o volume transferido e aproveita índices.

## Procedural não significa desorganizado

Um programa procedural pode ser bem projetado. Organização não depende apenas do paradigma, mas de decisões como:

- nomes claros;
- funções pequenas;
- limites de responsabilidade;
- controle de estado;
- tratamento de erros;
- separação entre regra e infraestrutura;
- testes automatizados;
- revisão do fluxo principal.

O problema não é executar passos. O problema é concentrar passos demais em uma função enorme, com estado escondido e efeitos colaterais difíceis de prever.

## Boas práticas

1. **Dê nomes que expressem ações.** Use nomes como `validarPedido` e `calcularTotal`.
2. **Mantenha funções pequenas.** Uma função deve realizar uma tarefa compreensível.
3. **Prefira dados locais.** Evite variáveis globais mutáveis.
4. **Passe dependências explicitamente.** Não esconda banco, rede ou configuração dentro de funções utilitárias.
5. **Separe cálculo de efeito colateral.** Calcule primeiro e grave ou envie depois quando isso deixar o fluxo mais seguro.
6. **Valide nas fronteiras.** Dados recebidos por uma [[Requisição]] não são confiáveis automaticamente.
7. **Trate erros de maneira clara.** Não ignore exceções nem retorne valores ambíguos.
8. **Evite loops com responsabilidades misturadas.** Extraia validação, cálculo e persistência quando necessário.
9. **Use early return com cuidado.** Retornar cedo para casos inválidos pode reduzir aninhamento e melhorar a leitura.
10. **Evite `goto` e fluxos difíceis de acompanhar.** Estruturas de sequência, decisão e repetição costumam ser suficientes.
11. **Escreva [[Testes]].** Funções pequenas e puras são especialmente fáceis de testar.
12. **Use [[Linting]].** Regras automáticas ajudam a encontrar variáveis não usadas, complexidade excessiva e padrões inconsistentes.
13. **Escolha o estilo que deixa o domínio mais claro.** Combine procedural, objetos e funções quando isso fizer sentido.

## Resumo

Programação procedural organiza o sistema como uma sequência de instruções, decisões, repetições e funções que transformam dados e estado. Ela é simples e útil para scripts, cálculos e fluxos bem definidos, e pode coexistir com orientação a objetos, programação funcional e frameworks. Para manter a qualidade, divida responsabilidades, evite estado global, trate erros, valide entradas e escreva testes.

### Veja também

- [[Linguagem de programação]]
- [[Java]]
- [[JavaScript]]
- [[TypeScript]]
- [[Backend]]
- [[Frontend]]
- [[React]]
- [[Next.js]]
- [[Node.js]]
- [[NestJS]]
- [[Quarkus]]
- [[ORM]]
- [[PostgreSQL]]
- [[Requisição]]
- [[Testes]]
- [[Linting]]
- [[Design Pattern]]
