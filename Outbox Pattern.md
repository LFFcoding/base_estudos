# Outbox Pattern

O **Outbox Pattern** (padrão outbox ou padrão da caixa de saída) é uma técnica para salvar uma alteração no banco de dados e registrar a mensagem que precisa ser enviada para outros sistemas dentro da mesma transação.

Depois, um processo separado lê os eventos registrados na tabela outbox e os publica em um [[Broker de fila]] ou em outro mecanismo de eventos.

## O problema das duas escritas

Imagine que o [[Backend]] precise:

1. criar um pedido no [[PostgreSQL]];
2. publicar `PedidoCriado` para o serviço de estoque.

São duas escritas em sistemas diferentes. Se a aplicação salvar o pedido e cair antes de publicar a mensagem, o estoque não será avisado. Se publicar a mensagem e falhar antes de salvar o pedido, o estoque poderá processar um pedido que não existe.

Esse problema é chamado de **dual write** (dupla escrita). Uma transação comum do PostgreSQL não consegue confirmar ou desfazer automaticamente uma publicação em um broker externo.

## Uma analogia

Imagine uma loja que vende um produto e precisa avisar o setor de entrega:

- a venda é registrada no sistema da loja;
- o aviso para a entrega é colocado em uma caixa de saída;
- ambos são registrados juntos no mesmo livro de operações;
- um funcionário lê a caixa de saída e leva os avisos ao setor de entrega;
- depois de entregar o aviso, marca a tarefa como enviada.

Se o funcionário faltar, o aviso continua na caixa. Se ele tentar entregar novamente, o setor precisa reconhecer que o aviso já foi processado.

## Como o padrão funciona

1. a aplicação inicia uma transação no banco;
2. grava a alteração principal, como um pedido;
3. grava um evento na tabela outbox;
4. confirma a transação;
5. um publicador procura eventos pendentes;
6. publica o evento no broker;
7. marca o evento como publicado;
8. se houver falha, tenta novamente mais tarde.

O ponto importante é que os dados principais e o evento ficam confirmados juntos. Se a transação falhar, os dois são desfeitos. Se o publicador falhar depois do commit, o evento continua disponível para uma nova tentativa.

## Estrutura da tabela outbox

Um exemplo simplificado em [[PostgreSQL]]:

```sql
CREATE TABLE outbox_events (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    payload JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    published_at TIMESTAMPTZ,
    attempts INTEGER NOT NULL DEFAULT 0
);
```

Os campos têm funções diferentes:

- `id`: identificador único do evento;
- `aggregate_type`: tipo da entidade, como `Pedido`;
- `aggregate_id`: identificador do pedido afetado;
- `event_type`: nome do acontecimento;
- `payload`: dados do evento, normalmente em [[JSON]];
- `created_at`: momento em que o evento foi criado;
- `published_at`: momento em que foi publicado com sucesso;
- `attempts`: quantidade de tentativas do publicador.

Em um sistema real, a tabela pode ter campos de prioridade, erro, versão, lote, destino e bloqueio temporário.

## Exemplo de gravação transacional

O serviço salva o pedido e o evento juntos:

```java
@Transactional
public Pedido criarPedido(NovoPedido dados) {
    var pedido = pedidoRepository.salvar(dados);

    outboxRepository.salvar(new OutboxEvent(
        "Pedido",
        pedido.id(),
        "PedidoCriado",
        "{\"pedidoId\":" + pedido.id() + "}"
    ));

    return pedido;
}
```

O exemplo é ilustrativo. Em código real, serialize o payload com uma biblioteca confiável e não monte JSON por concatenação quando os valores vierem de entrada externa.

No [[Quarkus]], a transação pode ser aplicada na camada de serviço conforme a configuração de persistência. O princípio não depende do framework: o pedido e o evento precisam usar a mesma transação e o mesmo banco.

## Exemplo de publicador

Um worker pode buscar eventos pendentes em pequenos lotes:

```sql
SELECT id, event_type, payload
FROM outbox_events
WHERE published_at IS NULL
ORDER BY created_at, id
LIMIT 100;
```

Depois, o publicador envia cada evento ao [[Broker de fila]]. Só deve marcar `published_at` depois de receber uma confirmação de publicação.

Em ambientes com vários publicadores, pode ser necessário reservar linhas para evitar que dois processos trabalhem no mesmo evento ao mesmo tempo. No PostgreSQL, uma estratégia comum usa `FOR UPDATE SKIP LOCKED`, mas a consulta deve ser projetada e testada conforme a transação e a duração do processamento.

## A mensagem pode ser duplicada

Mesmo com uma tabela outbox, pode acontecer:

1. o publicador envia o evento;
2. o broker recebe a mensagem;
3. o publicador cai antes de marcar `published_at`;
4. o mesmo evento é enviado novamente.

Por isso, o Outbox Pattern não garante sozinho exatamente uma publicação. O consumidor deve ser [[Idempotência|idempotente]], usando o `id` do evento, uma restrição única ou outra forma de deduplicação.

O processamento deve considerar entrega **at-least-once** (pelo menos uma vez). Isso é preferível a perder silenciosamente uma mensagem importante, mas exige consumidores preparados para repetições.

## Ordem dos eventos

Se um pedido for criado e logo depois cancelado, os consumidores precisam receber os eventos na ordem correta quando essa ordem fizer diferença.

Boas práticas incluem:

- guardar `created_at` e uma sequência por entidade;
- ordenar a leitura da outbox;
- usar uma chave de particionamento ou grupo por entidade no broker;
- não publicar um evento posterior enquanto um anterior da mesma entidade estiver pendente;
- documentar quando a ordem é ou não garantida.

Ordenar todos os eventos globalmente pode reduzir a escala. Muitas arquiteturas garantem ordem somente dentro de cada pedido, usuário ou partição.

## Retentativas e falhas

O publicador precisa distinguir falhas temporárias de permanentes:

- falha de rede: tentar novamente com espera progressiva;
- broker indisponível: manter o evento pendente;
- payload inválido: registrar erro e encaminhar para uma fila de análise;
- credencial inválida: alertar a equipe e impedir retries infinitos;
- consumidor rejeitando o evento: usar a política de retry e DLQ do broker.

Registre número de tentativas, último erro e próximo horário de tentativa. Não apague um evento apenas porque a primeira publicação falhou.

## Limpeza da outbox

Eventos publicados não precisam ficar para sempre na tabela, mas apagar cedo demais dificulta auditoria e reprocessamento.

Defina uma política:

- manter eventos por um período de auditoria;
- arquivar eventos antigos;
- remover somente depois de confirmação e retenção mínima;
- preservar os dados necessários para investigação;
- monitorar o tamanho da tabela e criar índices adequados.

Se a outbox crescer sem limite, ela pode ocupar o banco principal e prejudicar as operações da aplicação.

## Outbox e CDC

**CDC** (*Change Data Capture*) captura alterações realizadas no banco e as transforma em eventos. O Outbox Pattern grava explicitamente eventos de negócio em uma tabela; o CDC observa alterações e publica mudanças usando uma ferramenta ou recurso de captura.

Outbox costuma ser melhor quando a equipe quer controlar o nome, o formato e o momento do evento. CDC pode reduzir código da aplicação e aproveitar streams do banco, mas exige entender a ferramenta de captura, o formato das mudanças e sua operação.

Também existem outras opções:

- **transação distribuída**: tenta coordenar vários sistemas, mas aumenta complexidade e acoplamento;
- **saga**: coordena várias transações locais com ações de compensação;
- **event sourcing**: usa eventos como histórico principal das mudanças.

O Outbox Pattern resolve principalmente a publicação confiável de eventos relacionados a uma transação local. Ele não resolve todos os problemas de consistência entre serviços.

## Outbox na AWS

Na AWS, uma implementação pode usar Amazon RDS para a tabela outbox, um publicador e Amazon SQS como broker. A [orientação prescritiva da AWS sobre Transactional Outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html) descreve esse fluxo e destaca que consumidores de filas padrão devem ser idempotentes porque mensagens podem ser entregues mais de uma vez.

Quando a fonte é um banco NoSQL como DynamoDB, uma alternativa é usar Streams para capturar alterações e uma função Lambda para publicar os eventos em SQS ou EventBridge. Essa solução é diferente de uma tabela outbox relacional, mas atende ao mesmo objetivo de conectar alteração de dados e mensagem.

Em uma [[VPS]], a equipe pode executar o banco, o publicador e o broker em [[Docker Compose]], mas precisa cuidar de atualização, armazenamento, backups, monitoramento, retries e segurança. Serviços gerenciados da AWS reduzem parte do trabalho operacional, porém adicionam custos, limites e dependência do provedor.

## Segurança

- valide o payload antes de publicá-lo;
- não coloque senhas, tokens ou chaves privadas nos eventos;
- proteja a tabela e o broker com permissões mínimas;
- criptografe dados sensíveis em repouso e em trânsito;
- limite quem pode ler ou alterar a outbox;
- masque informações pessoais nos logs;
- trate eventos como dados que podem permanecer armazenados durante a retenção;
- use [[Segurança]] para orientar autenticação, autorização e proteção de segredos.

## Testes

Use [[Testes]] para verificar:

- pedido e outbox são confirmados juntos;
- se o pedido falha, o evento também não permanece confirmado;
- um publicador reiniciado retoma eventos pendentes;
- uma mensagem pode ser publicada novamente sem duplicar o efeito;
- eventos são publicados na ordem necessária;
- falhas permanentes vão para análise ou DLQ;
- a limpeza não remove eventos antes da retenção definida.

## Boas práticas

- grave eventos e dados principais na mesma transação;
- defina um `eventId` único e estável;
- faça consumidores idempotentes;
- publique em pequenos lotes e use retries com backoff;
- monitore eventos pendentes, idade, tentativas e erros;
- preserve ordenação somente onde ela for necessária;
- mantenha contratos de eventos versionados;
- planeje retenção, arquivamento e reprocessamento;
- não dependa de exatamente uma entrega;
- documente quem publica, quem consome e qual efeito cada evento representa.

**Outbox Pattern é uma técnica que registra a alteração principal e o evento de saída na mesma transação, permitindo publicar mensagens de forma confiável mesmo quando o broker ou a rede falham.**
