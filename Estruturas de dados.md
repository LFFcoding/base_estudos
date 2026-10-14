# O que são estruturas de dados?

**Estruturas de dados** (*data structures*) são formas de organizar informações na memória para que um programa consiga consultá-las, adicioná-las, remover itens ou percorrê-los.

Uma analogia: ferramentas diferentes organizam uma cozinha de maneiras diferentes. Uma gaveta deixa talheres fáceis de encontrar; uma pilha de pratos facilita pegar o prato de cima; uma agenda permite procurar alguém pelo nome. Cada estrutura ajuda com certos tipos de tarefa.

Uma estrutura influencia o custo das operações. A melhor escolha depende de como os dados serão usados, da quantidade de itens e das garantias de ordem que o programa precisa.

## Estrutura e operação

Ao escolher uma estrutura, pense nas operações mais importantes:

- **acessar:** obter um item;
- **buscar:** encontrar um valor;
- **inserir:** adicionar um valor;
- **remover:** apagar um valor;
- **percorrer:** visitar os itens;
- **ordenar:** organizar os itens por uma regra.

Nem toda estrutura faz tudo igualmente bem. Uma lista permite acessar itens por posição; um mapa encontra valores por uma chave; uma fila entrega primeiro o item mais antigo.

## Complexidade de tempo

A notação [[Big O]] descreve como o trabalho de uma operação tende a crescer quando a quantidade de dados aumenta. Ela não informa o tempo exato em segundos.

| Notação | Ideia aproximada | Exemplo comum |
| --- | --- | --- |
| `O(1)` | custo constante; não cresce com o tamanho da coleção | acessar uma posição de um array |
| `O(log n)` | cresce lentamente ao dividir o espaço de busca | buscar em uma árvore balanceada |
| `O(n)` | pode precisar olhar cada item | procurar um valor em uma lista sem índice |
| `O(n log n)` | comum em algoritmos eficientes de ordenação | Merge sort |
| `O(n²)` | compara muitos pares ou repete uma varredura para cada item | Bubble sort |

Esses valores dependem da estrutura e da operação. Uma busca em um mapa hash costuma ser `O(1)` em média, mas pode ser mais lenta em situações desfavoráveis. O [[Sorting|sorting]] também tem diferentes custos conforme o algoritmo.

## Array e lista

Um **array** guarda itens em posições numeradas. O acesso por índice costuma ser rápido. Em muitas linguagens, seu tamanho é fixo; outras estruturas oferecem uma lista que pode crescer.

Em Java, `ArrayList` é uma lista redimensionável baseada em array:

```java
List<String> culturas = new ArrayList<>();
culturas.add("milho");
culturas.add("soja");

String primeira = culturas.get(0);
```

O acesso por posição é rápido. Inserir ou remover no meio pode exigir deslocar outros itens. Para percorrer uma coleção, um laço simples costuma ser suficiente:

```java
for (String cultura : culturas) {
    System.out.println(cultura);
}
```

### Lista ligada (*linked list*)

Uma lista ligada conecta elementos chamados **nós (*nodes*)**. Cada nó guarda um valor e uma referência ao próximo nó; em uma lista duplamente ligada, também pode apontar para o anterior.

Ela pode facilitar inserções e remoções em posições cujo nó já foi encontrado, mas acessar o item de índice 500 exige percorrer os nós anteriores. Para a maioria dos usos comuns em Java, `ArrayList` costuma ser uma escolha mais simples e eficiente em memória e acesso. Escolha uma lista ligada quando seu padrão de operações justificar essa estrutura.

## Pilha

Uma **pilha (*stack*)** segue a regra [[FIFO e LIFO|LIFO]] (*Last In, First Out*): o último item colocado é o primeiro a sair.

É como uma pilha de livros: para pegar um livro que está embaixo, primeiro é necessário retirar os que estão acima. Pilhas aparecem em histórico de desfazer, chamadas de funções e análise de expressões.

Exemplo com `ArrayDeque` em Java:

```java
Deque<String> historico = new ArrayDeque<>();
historico.push("tela inicial");
historico.push("detalhe do pedido");

String telaAnterior = historico.pop(); // "detalhe do pedido"
```

`push` coloca no topo e `pop` remove o item do topo. Verifique se a estrutura está vazia antes de remover quando isso for possível.

## Fila e deque

Uma **fila (*queue*)** segue a regra [[FIFO e LIFO|FIFO]] (*First In, First Out*): o primeiro item que entra é o primeiro a sair. É como uma fila de atendimento.

```java
Queue<String> atendimentos = new ArrayDeque<>();
atendimentos.offer("cliente A");
atendimentos.offer("cliente B");

String proximo = atendimentos.poll(); // "cliente A"
```

`offer` adiciona ao fim e `poll` retira do início. A estrutura de dados fila se conecta ao conceito de [[Filas]] usado em sistemas de mensageria, embora uma fila em memória e um broker de mensagens ofereçam garantias diferentes.

Um **deque** (*double-ended queue*) permite inserir ou remover itens nas duas pontas. Ele pode servir tanto como fila quanto como pilha.

## Conjunto

Um **conjunto (*set*)** guarda valores sem duplicatas. Ele é útil para verificar se um item já foi visto ou garantir que cada identificador apareça uma única vez.

```java
Set<String> sensoresAtivos = new HashSet<>();
sensoresAtivos.add("sensor-12");
sensoresAtivos.add("sensor-12"); // não cria um segundo valor igual

boolean ativo = sensoresAtivos.contains("sensor-12");
```

Um `HashSet` não garante uma ordem específica ao percorrer os itens. Se a ordem for necessária, escolha uma implementação que a preserve ou ordene os valores explicitamente.

## Mapa ou dicionário

Um **mapa (*map*)** associa uma chave a um valor. Em Java, `Map` é uma interface; `HashMap` é uma implementação comum.

```java
Map<String, Integer> leiturasPorSensor = new HashMap<>();
leiturasPorSensor.put("sensor-12", 31);

Integer umidade = leiturasPorSensor.get("sensor-12");
```

O mapa permite buscar um valor pela chave sem percorrer manualmente toda a coleção. `HashMap` não garante a ordem de iteração; use uma implementação ordenada se a ordem fizer parte do requisito.

Mapas são usados em contagens, índices em memória, configurações e caches. Um mapa em memória desaparece quando o processo termina, então não substitui um banco de dados persistente como [[PostgreSQL]].

## Árvores

Uma **árvore (*tree*)** organiza nós em uma hierarquia. Há um nó inicial, chamado **raiz (*root*)**, e relações entre pais e filhos.

Exemplos do cotidiano:

- pastas e subpastas;
- categorias de produtos;
- hierarquia de uma organização;
- elementos de uma página web.

Uma **árvore de busca binária** organiza valores para permitir buscas e inserções. Quando está balanceada, essas operações podem ser `O(log n)`. Se ficar muito desequilibrada, pode se aproximar de `O(n)`. Árvores balanceadas, como AVL e Red-Black, controlam esse crescimento.

## Heap e fila de prioridade

Um **heap** é uma árvore especializada que permite encontrar rapidamente o item de maior ou menor prioridade. Ele costuma ser usado para implementar uma **fila de prioridade (*priority queue*)**.

Exemplo: um sistema pode processar primeiro alertas críticos e depois alertas informativos. A fila de prioridade escolhe o próximo item pela prioridade, não apenas pela ordem de chegada.

Em Java, a classe `PriorityQueue` oferece essa estrutura. A ordem de iteração da coleção não deve ser tratada como uma lista totalmente ordenada; use as operações próprias da fila para obter o próximo elemento.

## Grafo

Um **grafo (*graph*)** representa elementos e as conexões entre eles:

- **vértice (*vertex*):** um elemento;
- **aresta (*edge*):** uma conexão entre elementos.

Um mapa de estradas pode ser representado como um grafo: cidades são vértices e estradas são arestas. Grafos também representam dependências de módulos, conexões sociais e rotas de comunicação.

As conexões podem ter direção e custo. Um algoritmo pode procurar uma rota, identificar componentes conectados ou detectar ciclos.

## Como escolher uma estrutura?

| Necessidade principal | Estrutura que pode servir |
| --- | --- |
| Acessar itens por posição | array ou lista baseada em array |
| Processar primeiro o item que chegou | fila |
| Acessar o item adicionado mais recentemente | pilha |
| Evitar valores repetidos | conjunto |
| Encontrar um valor pela chave | mapa |
| Representar hierarquia | árvore |
| Buscar o próximo item por prioridade | heap ou fila de prioridade |
| Representar relações e caminhos | grafo |

Essa tabela é um ponto de partida. Considere também o volume de dados, a ordem necessária, a frequência de leitura e escrita, a memória disponível e se os dados precisam sobreviver ao encerramento do programa.

## Estruturas em memória e estruturas persistentes

Listas, mapas e conjuntos da linguagem normalmente vivem na memória do processo. São rápidos para o trabalho atual, mas podem desaparecer se a aplicação parar.

Já uma tabela em [[Banco de dados]] guarda dados de forma persistente. O banco também usa estruturas internas, como índices, para acelerar consultas. Um índice melhora certas buscas, mas consome espaço e pode tornar gravações mais trabalhosas.

Não carregue um conjunto enorme inteiro na memória apenas para encontrar alguns registros. Use consultas e índices do banco quando os dados forem persistentes e volumosos.

## Boas práticas

- escolha a estrutura conforme as operações mais frequentes;
- entenda as garantias de ordem e duplicidade;
- considere custo de tempo e memória;
- use as estruturas da biblioteca padrão antes de implementar uma do zero;
- não suponha que um mapa ou conjunto mantém ordem sem consultar o contrato da implementação;
- valide limites e entradas antes de adicionar dados;
- pense em concorrência quando várias tarefas alterarem a mesma estrutura;
- use banco de dados para persistência, em vez de confiar em coleções em memória;
- escreva [[Testes]] para casos vazios, valores repetidos, limites e remoções;
- meça o desempenho com dados representativos antes de otimizar.

## Resumo

> **Estruturas de dados organizam informações para que operações como buscar, inserir e remover sejam feitas de forma adequada ao problema.**

Arrays, listas, pilhas, filas, conjuntos, mapas, árvores, heaps e grafos resolvem necessidades diferentes. Entender suas propriedades ajuda a escrever programas mais claros e eficientes.
