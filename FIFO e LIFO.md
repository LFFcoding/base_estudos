# O que significam FIFO e LIFO?

**FIFO** (*First In, First Out*) significa **primeiro a entrar, primeiro a sair**. **LIFO** (*Last In, First Out*) significa **último a entrar, primeiro a sair**. São regras que descrevem a ordem em que os itens são retirados de uma coleção.

Uma analogia simples:

- **FIFO:** uma fila de pessoas; quem chegou primeiro normalmente é atendido primeiro;
- **LIFO:** uma pilha de pratos; o prato colocado por último fica em cima e é retirado primeiro.

FIFO e LIFO não são linguagens ou ferramentas. São políticas de ordenação usadas por estruturas de dados e sistemas.

## FIFO: primeiro a entrar, primeiro a sair

Imagine que os itens `A`, `B` e `C` entram nesta ordem:

```text
Entrada: A → B → C
Saída:   A → B → C
```

Uma fila costuma seguir FIFO. Em Java, uma fila em memória pode ser usada assim:

```java
Queue<String> fila = new ArrayDeque<>();
fila.offer("A");
fila.offer("B");
fila.offer("C");

String primeiro = fila.poll(); // "A"
```

`offer` adiciona um item ao fim da fila e `poll` retira o próximo do início. Se estiver vazia, `poll` retorna `null`.

Esse modelo é útil para tarefas que devem ser tratadas pela ordem de chegada, como trabalhos simples de impressão ou processamento sequencial. Em sistemas de mensageria, a ordem real depende das garantias do broker, do número de consumidores, das tentativas e da configuração.

## LIFO: último a entrar, primeiro a sair

Imagine que os itens `A`, `B` e `C` sejam empilhados nesta ordem:

```text
Entrada: A → B → C
Saída:   C → B → A
```

Uma pilha costuma seguir LIFO. Em Java, `Deque` pode ser usada como pilha:

```java
Deque<String> pilha = new ArrayDeque<>();
pilha.push("A");
pilha.push("B");
pilha.push("C");

String ultimo = pilha.pop(); // "C"
```

`push` coloca um item no topo e `pop` retira o item do topo. `pop` lança uma exceção se a pilha estiver vazia; use `isEmpty()` antes quando a coleção puder estar sem itens.

LIFO aparece em chamadas de funções, histórico de desfazer/refazer e análise de expressões. Quando uma função chama outra, a chamada mais recente normalmente termina antes de a execução voltar às chamadas anteriores.

## Comparação

| Regra | O que sai primeiro? | Estrutura comum | Analogia |
| --- | --- | --- | --- |
| FIFO | o item que entrou há mais tempo | fila (*queue*) | fila de atendimento |
| LIFO | o item que entrou mais recentemente | pilha (*stack*) | pilha de pratos |

No estudo de [[Estruturas de dados]], FIFO se relaciona a filas e LIFO a pilhas. O conceito de [[Filas]] também aparece em brokers e processamento de mensagens.

## FIFO não significa “uma única entrega”

FIFO descreve **ordem**. Termos como *at-most-once* e *at-least-once* descrevem **possibilidade de perda ou repetição**. São propriedades diferentes.

Uma mensagem pode ser entregue em ordem FIFO e ainda ser entregue novamente após uma falha. Por isso, consumidores de [[Filas]] precisam considerar mensagens duplicadas e podem precisar de [[Idempotência]].

Além disso, múltiplos consumidores podem terminar o trabalho em uma ordem diferente da ordem de retirada. Alguns brokers garantem ordem apenas dentro de um grupo, partição ou chave, e uma mensagem repetida pode afetar o fluxo. Consulte as garantias específicas do serviço utilizado.

## Prioridade pode mudar a ordem

Uma fila de prioridade pode retirar primeiro o item com maior prioridade, mesmo que ele tenha chegado depois. Nesse caso, a regra principal não é FIFO, embora itens com a mesma prioridade possam ter uma regra adicional de desempate.

Defina a política adequada ao problema:

- se a justiça por ordem de chegada for necessária, use uma fila FIFO com garantias apropriadas;
- se urgências precisarem passar na frente, use prioridade e documente esse comportamento;
- se o item mais recente tiver de ser resolvido primeiro, uma pilha LIFO pode fazer sentido.

## Boas práticas

- escreva explicitamente qual regra de remoção o sistema precisa;
- não suponha que uma fila de broker preserva ordem em qualquer configuração;
- se a ordem importa, teste o comportamento com concorrência, retry e falhas;
- use identificadores e processamento [[Idempotência|idempotente]] para lidar com reentregas;
- escolha fila, pilha ou prioridade conforme o significado do trabalho;
- explique a política no contrato do sistema para que consumidores saibam o que esperar.

## Resumo

> **FIFO retira primeiro o item mais antigo; LIFO retira primeiro o item mais recente.**

FIFO costuma ser associado a filas e LIFO a pilhas. Em sistemas distribuídos, ordem não garante entrega única nem conclusão na mesma ordem; essas garantias dependem do broker e da configuração.
