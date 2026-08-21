# O que é método de sorting?

*Sorting* significa **ordenação**. Um método de sorting é uma forma de organizar os itens de uma lista de acordo com uma regra.

Por exemplo, uma lista de notas pode ser ordenada da menor para a maior, uma lista de nomes pode ser organizada em ordem alfabética e uma lista de produtos pode ser organizada pelo preço.

Uma analogia simples é organizar cartas de baralho: você pode colocá-las em ordem pelo número, pelo naipe ou por algum outro critério. O método de sorting é a estratégia usada para fazer essa organização.

## Ordem crescente e decrescente

- **Crescente (*ascending*):** do menor para o maior, como `1, 2, 3`.
- **Decrescente (*descending*):** do maior para o menor, como `3, 2, 1`.
- **Ordem alfabética:** normalmente, de `A` a `Z`.
- **Chave de ordenação (*sort key*):** o campo usado para comparar os itens, como `nome`, `preco` ou `data`.

Também é possível usar mais de uma chave. Por exemplo: ordenar pessoas por cidade e, quando a cidade for igual, por nome.

## Algoritmos conhecidos

Um algoritmo de sorting é uma sequência de passos que transforma uma lista desorganizada em uma lista ordenada.

| Algoritmo | Ideia principal | Complexidade comum | Quando estudar ou usar |
|---|---|---:|---|
| **Bubble sort** | Compara vizinhos e troca os que estão fora de ordem. | `O(n²)` | Bom para aprender, mas geralmente ruim para listas grandes. |
| **Selection sort** | Procura o menor item e o coloca na próxima posição. | `O(n²)` | Simples de entender, mas pouco eficiente em muitos dados. |
| **Insertion sort** | Insere cada item no lugar correto entre os anteriores. | `O(n²)` | Pode funcionar bem em listas pequenas ou quase ordenadas. |
| **Merge sort** | Divide a lista, ordena as partes e depois as intercala. | `O(n log n)` | Bom desempenho e ordenação estável, usando memória extra. |
| **Quicksort** | Escolhe um pivô e separa itens menores e maiores. | `O(n log n)` em média | Muito eficiente em muitos cenários; a escolha do pivô importa. |
| **Heap sort** | Usa uma estrutura de heap para escolher o próximo item. | `O(n log n)` | Oferece desempenho previsível, mas costuma ser menos simples. |

`n` representa a quantidade de itens. Em geral, `O(n log n)` cresce melhor que `O(n²)` quando a lista fica grande.

Na prática, normalmente é melhor usar a função de ordenação da linguagem ou do banco de dados. Os algoritmos são importantes para entender o custo das escolhas e para situações especiais.

## Exemplo em Java

Em uma aplicação de [[Backend]] feita com [[Java]], podemos ordenar uma lista usando `List.sort` e um `Comparator`:

```java
var nomes = new ArrayList<>(List.of("Carlos", "Ana", "Bruno"));

nomes.sort(Comparator.naturalOrder());

System.out.println(nomes); // [Ana, Bruno, Carlos]
```

Para ordenar do maior para o menor:

```java
var notas = new ArrayList<>(List.of(7.5, 9.0, 6.0));

notas.sort(Comparator.reverseOrder());

System.out.println(notas); // [9.0, 7.5, 6.0]
```

Para objetos, informamos qual propriedade será usada:

```java
produtos.sort(Comparator.comparing(Produto::getPreco));
```

Esse código ordena `produtos` pelo preço. Para desempatar pelo nome, podemos combinar comparadores:

```java
produtos.sort(
    Comparator.comparing(Produto::getPreco)
        .thenComparing(Produto::getNome)
);
```

## Exemplo em JavaScript e TypeScript

Em [[JavaScript]] e [[TypeScript]], o método `sort` altera a própria lista. Para números, devemos informar uma função de comparação:

```ts
const numeros = [10, 2, 30];

numeros.sort((a, b) => a - b);

console.log(numeros); // [2, 10, 30]
```

Sem a função `(a, b) => a - b`, o JavaScript pode comparar os números como textos e produzir uma ordem inesperada, como `10, 2, 30`.

Para ordenar objetos:

```ts
type Produto = {
  nome: string;
  preco: number;
};

const produtos: Produto[] = [
  { nome: "Caderno", preco: 25 },
  { nome: "Caneta", preco: 5 },
];

produtos.sort((a, b) => a.preco - b.preco);
```

Em uma tela de [[Frontend]], como uma aplicação com [[Next.js]], essa técnica pode ordenar poucos itens já carregados. Para muitos itens, o ideal costuma ser pedir ao [[Backend]] os dados ordenados e paginados.

## Ordenação no banco de dados

Quando os dados estão em um [[Banco de dados]], o servidor do banco geralmente consegue ordenar com mais eficiência do que a aplicação, principalmente quando há muitos registros.

No [[PostgreSQL]], usamos `ORDER BY`:

```sql
SELECT id, nome, criado_em
FROM usuarios
ORDER BY criado_em DESC, id DESC
LIMIT 20;
```

Esse exemplo busca os 20 usuários mais recentes. `id DESC` funciona como um segundo critério para deixar a ordem previsível quando dois registros possuem a mesma data.

Com um [[ORM]], como [[Prisma]], a ordenação pode ser solicitada no código e transformada em uma consulta para o banco:

```ts
const usuarios = await prisma.usuario.findMany({
  orderBy: [
    { criadoEm: "desc" },
    { id: "desc" },
  ],
  take: 20,
});
```

## Ordenação estável

Uma ordenação é **estável (*stable sort*)** quando itens com a mesma chave mantêm a ordem que tinham antes.

Imagine uma lista já ordenada por nome e depois ordenada por cidade. Se duas pessoas moram na mesma cidade, uma ordenação estável preserva a ordem dos nomes entre elas.

Mesmo quando a ferramenta é estável, é uma boa prática definir critérios de desempate explícitos, como `id`. Assim, páginas diferentes não ficam mudando de ordem enquanto o usuário navega.

## Ordenar na aplicação ou no banco?

Uma regra prática:

- **Poucos itens já carregados:** a aplicação pode ordenar em memória.
- **Muitos itens:** deixe o banco ordenar e use paginação.
- **Dados exibidos em páginas:** ordene antes de aplicar `LIMIT` e `OFFSET`, ou use uma estratégia de paginação adequada.
- **Ordenação por campo escolhido pelo usuário:** aceite apenas campos permitidos.

Carregar milhares de registros para ordenar tudo na aplicação gasta memória, aumenta o tempo de resposta e pode deixar o sistema lento.

## Boas práticas

1. **Defina claramente o critério.** Diga se a ordem é crescente ou decrescente e qual campo será usado.
2. **Use desempate.** Combine a chave principal com um identificador único, como `id`.
3. **Use a ferramenta adequada.** Prefira os métodos nativos da linguagem e o `ORDER BY` do banco a implementar um algoritmo manual sem necessidade.
4. **Pense na complexidade.** Evite `O(n²)` para listas grandes quando uma solução `O(n log n)` ou a ordenação do banco estiver disponível.
5. **Trate valores vazios.** Defina se `null` deve aparecer primeiro ou por último.
6. **Padronize textos.** Decida como lidar com maiúsculas, minúsculas, acentos e locale.
7. **Proteja campos dinâmicos.** Nunca monte um `ORDER BY` diretamente com texto recebido do usuário. Use uma lista de campos permitidos para evitar injeção de SQL.
8. **Combine com índices e paginação.** Em consultas frequentes, um índice compatível com o filtro e a ordenação pode ajudar o banco.
9. **Teste casos de borda.** Verifique lista vazia, itens repetidos, valores nulos, caracteres especiais e números negativos.

## Sorting não é filtragem

Essas operações são diferentes:

- **Filtrar (*filter*):** escolhe quais itens continuam na lista.
- **Ordenar (*sort*):** define em qual sequência os itens aparecem.

Por exemplo, em uma busca de produtos, podemos filtrar apenas produtos disponíveis e depois ordená-los pelo menor preço.

## Resumo

Método de sorting é uma estratégia para colocar dados em uma ordem definida. Para aprender, vale conhecer Bubble sort, Insertion sort, Merge sort e Quicksort. Para sistemas reais, prefira recursos nativos, ordenação no banco quando os dados forem numerosos, critérios de desempate e validação dos campos recebidos.

### Veja também

- [[Backend]]
- [[Banco de dados]]
- [[PostgreSQL]]
- [[Java]]
- [[JavaScript]]
- [[TypeScript]]
- [[ORM]]
- [[Prisma]]
- [[Testes]]
