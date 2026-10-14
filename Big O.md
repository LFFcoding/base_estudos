# O que é Big O?

**Big O** é uma notação usada para descrever como o custo de um algoritmo cresce quando aumenta a quantidade de dados de entrada.

Uma analogia: imagine procurar uma pessoa em uma lista telefônica. Se a lista não estiver organizada, talvez seja preciso olhar muitos nomes. Se estiver ordenada e você puder descartar metade a cada consulta, encontrará a pessoa com menos passos. Big O ajuda a comparar esse crescimento.

Big O não prevê o tempo exato em segundos. O tempo real também depende do computador, da linguagem, da implementação e dos dados. A notação ajuda a entender como o trabalho tende a crescer para entradas maiores.

## O que é `n`?

`n` representa o tamanho da entrada. Dependendo do problema, pode significar:

- quantidade de itens em uma lista;
- número de caracteres em um texto;
- quantidade de linhas em uma tabela;
- quantidade de vértices ou conexões em um grafo.

Ao analisar um algoritmo, deixe claro o que `n` representa. Em algoritmos com duas entradas, podem existir dois tamanhos, como `n` e `m`.

## Crescimentos comuns

| Notação | Nome informal | Exemplo intuitivo |
| --- | --- | --- |
| `O(1)` | constante | acessar uma posição por índice |
| `O(log n)` | logarítmico | dividir repetidamente o espaço de busca ao meio |
| `O(n)` | linear | percorrer todos os itens uma vez |
| `O(n log n)` | linearítmico | muitos algoritmos eficientes de ordenação |
| `O(n²)` | quadrático | comparar cada item com todos os demais |
| `O(2ⁿ)` | exponencial | explorar todas as combinações possíveis de `n` escolhas |
| `O(n!)` | fatorial | testar todas as ordens possíveis de `n` elementos |

Para entradas grandes, os crescimentos exponencial e fatorial tendem a ficar impraticáveis rapidamente. Mesmo assim, o melhor algoritmo depende do problema e dos limites reais.

## Exemplos de análise

### Tempo constante: `O(1)`

```java
int primeiro = valores.get(0);
```

Se a lista permite acesso por índice, essa operação não precisa percorrer todos os itens. A quantidade de itens não muda a ordem de crescimento desse acesso.

### Tempo linear: `O(n)`

```java
boolean encontrou = false;

for (int valor : valores) {
    if (valor == procurado) {
        encontrou = true;
        break;
    }
}
```

No pior caso, o algoritmo olha todos os `n` valores. Se o item estiver no começo, ele pode terminar antes; a análise precisa declarar qual cenário está considerando.

### Tempo quadrático: `O(n²)`

```java
for (int i = 0; i < valores.size(); i++) {
    for (int j = 0; j < valores.size(); j++) {
        comparar(valores.get(i), valores.get(j));
    }
}
```

Para cada item, o exemplo percorre a coleção inteira. O número aproximado de comparações cresce como `n × n`. Esse tipo de custo pode ser aceitável para dados pequenos, mas pesa conforme a entrada aumenta.

### Tempo logarítmico: `O(log n)`

Uma busca binária procura em uma coleção ordenada e descarta aproximadamente metade dos itens a cada passo:

```text
1000 itens → 500 → 250 → 125 → ...
```

É necessário que os dados estejam ordenados e que a estrutura permita acessar os pontos intermediários com eficiência. Fazer busca binária em uma lista ligada não oferece necessariamente o mesmo benefício.

### Tempo `O(n log n)`

Algoritmos como Merge sort normalmente dividem os dados em partes menores, ordenam essas partes e depois as combinam. A complexidade é adequada para muitos conjuntos grandes. Veja [[Sorting]] para comparar algoritmos de ordenação.

## Regras práticas para analisar código

Essas regras ajudam a fazer uma estimativa, mas não substituem uma análise cuidadosa:

- uma sequência de operações `O(n)` seguida de outra `O(n)` continua sendo `O(n)`;
- dois laços independentes de tamanho `n` somam `O(n) + O(n)`, que simplifica para `O(n)`;
- um laço dentro de outro, ambos de tamanho `n`, costuma resultar em `O(n²)`;
- se um laço reduz o espaço pela metade a cada repetição, pode resultar em `O(log n)`;
- se cada repetição cria duas novas chamadas recursivas, o custo pode crescer exponencialmente;
- quando os tamanhos de entrada diferem, preserve variáveis diferentes, como `O(n × m)`.

Big O normalmente omite fatores constantes e termos menores porque descreve a tendência para entradas grandes:

```text
3n² + 5n + 8  →  O(n²)
```

Isso não significa que as constantes nunca importem. Para listas pequenas, um algoritmo `O(n²)` simples pode ser mais rápido ou mais fácil de manter que um algoritmo `O(n log n)` complexo. Big O descreve crescimento, não todas as diferenças de execução.

## Melhor caso, caso médio e pior caso

Uma análise pode considerar diferentes entradas:

- **melhor caso (*best case*):** cenário mais favorável;
- **caso médio (*average case*):** custo esperado para uma distribuição de entradas definida;
- **pior caso (*worst case*):** limite para o cenário mais desfavorável.

Por exemplo, uma busca linear encontra o item na primeira posição em `O(1)` no melhor caso e pode percorrer todos os elementos, chegando a `O(n)` no pior caso.

Quando alguém diz apenas “o algoritmo é `O(n)`”, é útil esclarecer se está falando do pior caso, do caso médio ou de outro limite. Big O é uma notação de limite assintótico; não significa, por definição, “pior caso”, embora muitas análises de algoritmos apresentem o pior caso usando Big O.

## Complexidade de memória

Também podemos estimar quanto espaço extra um algoritmo usa. Isso é chamado de **complexidade espacial (*space complexity*)**.

```java
List<Integer> positivos = new ArrayList<>();

for (int valor : valores) {
    if (valor > 0) {
        positivos.add(valor);
    }
}
```

No pior caso, a nova lista guarda todos os `n` itens, então usa `O(n)` de espaço adicional. Uma soma que guarda apenas um acumulador pode usar `O(1)` de espaço adicional.

É importante dizer se a análise inclui a entrada original ou se conta apenas a memória extra. Muitas comparações de algoritmos usam espaço auxiliar para mostrar o custo adicional.

## Custo amortizado

Algumas operações são baratas quase sempre, mas ocasionalmente fazem um trabalho maior. Uma lista redimensionável, por exemplo, pode precisar copiar seus itens para uma área maior quando a capacidade acaba.

O **custo amortizado (*amortized cost*)** considera o custo médio por operação ao longo de uma sequência, sem depender de uma distribuição aleatória. Adicionar um elemento a uma lista dinâmica costuma ser `O(1)` amortizado, mesmo que uma expansão individual possa custar `O(n)`.

Isso ajuda a evitar concluir que toda inserção é cara por causa de uma expansão rara.

## Estrutura de dados e Big O

A complexidade depende da combinação entre operação e estrutura escolhida:

| Operação aproximada | Estrutura ou condição | Custo comum |
| --- | --- | --- |
| acesso por índice | array ou `ArrayList` | `O(1)` |
| procurar um item | lista sem índice | `O(n)` |
| buscar uma chave | `HashMap`, em média | `O(1)` |
| buscar em árvore balanceada | árvore de busca | `O(log n)` |
| inserir no início de array dinâmico | pode deslocar itens | `O(n)` |
| inserir e remover nas pontas | deque | `O(1)` em implementações comuns |

Os detalhes dependem da implementação e do caso. [[Estruturas de dados]] explica as propriedades de listas, pilhas, filas, mapas, árvores e grafos.

## Big O e bancos de dados

Uma consulta em um [[Banco de dados]] pode exigir a leitura de muitos registros. Um índice pode acelerar certas buscas, mas ocupa espaço e adiciona trabalho às gravações. Portanto, criar índices para todas as colunas não é uma solução automática.

Na prática, examine as consultas reais, o plano de execução, o tamanho dos dados e os índices. Big O ajuda a raciocinar, mas o desempenho também depende de disco, memória, cache, concorrência e configuração do banco.

## Como usar Big O com bom senso

- defina o que `n` representa;
- identifique as operações que dominam o trabalho;
- analise tempo e memória separadamente;
- considere melhor, médio e pior caso quando isso mudar a decisão;
- observe as operações da estrutura de dados usada;
- não compare algoritmos sem considerar implementação e tamanho real da entrada;
- meça com dados representativos quando desempenho for importante;
- escreva [[Testes]] para garantir que uma otimização preserve o comportamento;
- prefira código simples até haver evidência de um gargalo.

O [[Backend]] também pode ficar lento por causa de rede, banco, bloqueios ou espera por outro serviço. Nem todo problema de desempenho é causado pela complexidade do algoritmo.

## Resumo

> **Big O descreve como o tempo ou a memória usados por um algoritmo tendem a crescer conforme aumenta a entrada.**

É uma ferramenta para comparar soluções e prever gargalos em escala. Use-a junto com o contexto, os testes e as medições do sistema real.
