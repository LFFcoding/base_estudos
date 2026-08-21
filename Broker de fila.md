# Broker de fila

Um **broker de fila** (*message broker*) é um serviço intermediário que recebe, organiza, armazena e entrega mensagens entre partes de um sistema. Ele é a peça central de muitos fluxos descritos em [[Filas]].

Ele permite que o produtor de um trabalho não precise conhecer diretamente o consumidor. O produtor envia a mensagem ao broker, e o broker decide como guardá-la e para quem entregá-la.

## Uma analogia

Imagine uma central de distribuição:

- uma loja envia uma encomenda para a central;
- a central identifica o destino;
- a encomenda fica armazenada até poder seguir viagem;
- o entregador recebe a encomenda;
- a central registra se a entrega foi confirmada ou se precisa ser tentada novamente.

O broker faz esse papel com mensagens. Ele ajuda os sistemas a trabalhar em ritmos diferentes e a continuar funcionando mesmo quando um consumidor está temporariamente indisponível.

## Fila e broker não são a mesma coisa

- **Fila (*queue*)**: lugar onde mensagens aguardam processamento, normalmente na ordem definida pela configuração.
- **Broker**: serviço que pode administrar filas, tópicos, roteamento, consumidores, confirmações, retries, persistência e segurança.

Uma fila é como uma fila de atendimento. O broker é a central que cria as filas, distribui senhas, chama atendentes e registra problemas.

Uma aplicação pode usar uma estrutura simples do [[Redis]] como fila, mas um broker dedicado, como RabbitMQ ou Amazon MQ, costuma oferecer mais recursos de roteamento e operação. A escolha depende do fluxo, da escala e dos [[Protocolos de mensageria|protocolos de mensageria]] usados pela equipe.

## Participantes

- **Produtor (*producer*)**: publica uma mensagem.
- **Broker**: recebe, armazena e encaminha a mensagem.
- **Fila (*queue*)**: guarda mensagens para um grupo de consumidores.
- **Consumidor (*consumer*)**: recebe e processa a mensagem.
- **Worker**: processo que executa o trabalho do consumidor.
- **Mensagem (*message*)**: dados que descrevem uma tarefa ou acontecimento, muitas vezes em [[JSON]].

Exemplo de mensagem:

```json
{
  "eventId": "evento-123",
  "tipo": "EnviarEmail",
  "usuarioId": 42
}
```

O `eventId` ajuda o consumidor a detectar mensagens repetidas e aplicar [[Idempotência]].

## Fluxo básico

1. o [[Backend]] recebe uma [[Requisição]];
2. o backend valida a entrada e salva o estado principal;
3. o produtor envia uma mensagem ao broker;
4. o broker guarda a mensagem em uma fila;
5. um worker consulta a fila;
6. o broker entrega uma mensagem ao worker;
7. o worker executa o trabalho;
8. o worker confirma o processamento;
9. o broker remove a mensagem ou a encaminha conforme o resultado.

Se o worker parar antes da confirmação, o broker pode disponibilizar a mensagem novamente. Isso evita perda, mas pode gerar processamento repetido. O consumidor precisa ser idempotente.

## Confirmação (*acknowledgement*)

O **ack** é a confirmação de que o consumidor terminou o trabalho.

Sem ack, o broker pode considerar uma mensagem entregue mesmo que o worker tenha falhado. Com ack manual, a mensagem permanece disponível até o consumidor confirmar.

O consumidor deve confirmar somente depois de concluir as etapas importantes. Confirmar antes de salvar o resultado pode fazer o trabalho desaparecer quando o processo falhar logo depois.

## Rejeição, retry e DLQ

Quando o processamento falha, o consumidor ou o broker pode:

- tentar novamente imediatamente;
- esperar antes de tentar de novo, usando *backoff*;
- devolver a mensagem à fila;
- rejeitar a mensagem;
- encaminhar a mensagem para uma **DLQ** (*dead-letter queue*).

Uma DLQ guarda mensagens que falharam muitas vezes ou que não podem ser processadas. Ela ajuda a investigar o problema sem bloquear todas as outras mensagens.

Não faça retry infinito e imediato. Uma mensagem inválida pode consumir recursos continuamente. Defina limite de tentativas, motivo da falha, retenção e procedimento de correção.

## Roteamento

Um broker pode entregar mensagens de formas diferentes:

- **fila direta**: a mensagem vai para uma fila específica;
- **fan-out**: uma mensagem é copiada para várias filas interessadas;
- **por tópico**: consumidores recebem mensagens de assuntos que assinaram;
- **por chave de roteamento**: regras escolhem a fila conforme um campo ou padrão;
- **por prioridade**: mensagens importantes podem ser processadas antes de outras, se o broker suportar essa política.

Em [[RabbitMQ]], por exemplo, [[Exchanges e bindings|exchanges]] recebem mensagens e usam bindings para encaminhá-las às filas. Em serviços com tópicos, um publicador envia para o assunto e várias filas ou consumidores podem receber uma cópia.

## Fila, pub/sub e stream

- **Fila**: normalmente uma mensagem é processada por um consumidor de um grupo;
- **pub/sub**: vários assinantes recebem uma mensagem publicada em um canal ou tópico;
- **stream**: eventos ficam registrados em uma sequência que pode ser lida por consumidores e grupos diferentes.

O modelo escolhido depende de a mensagem representar um trabalho único, um aviso para várias partes ou um histórico de eventos que precisa ser reprocessado.

## Entrega e duplicidade

Um broker pode oferecer diferentes garantias:

- **at-most-once**: pode perder mensagens, mas tenta não repetir;
- **at-least-once**: tenta não perder mensagens, mas pode entregar a mesma mensagem mais de uma vez;
- **FIFO**: preserva a ordem em determinadas condições;
- **deduplicação**: identifica mensagens repetidas em uma janela ou configuração específica.

“Exatamente uma vez” é difícil em sistemas distribuídos. Mesmo que o broker faça uma entrega controlada, o consumidor pode cair depois de executar uma operação e antes de enviar o ack. Use [[Idempotência]], identificadores únicos e restrições no banco.

## Broker e PostgreSQL

O broker não substitui o [[PostgreSQL]]. Uma divisão comum é:

- PostgreSQL: guarda usuários, pedidos e o estado principal;
- broker: entrega trabalhos e eventos entre componentes;
- worker: executa tarefas assíncronas;
- [[Backend]]: recebe requisições, aplica regras e publica mensagens.

Para evitar salvar o pedido e perder a mensagem em uma segunda operação, considere o [[Outbox Pattern]]. O pedido e o evento pendente são gravados na mesma transação; um publicador envia o evento ao broker depois.

## Exemplos de tecnologias

- **[[RabbitMQ]]**: broker tradicional com filas, [[Exchanges e bindings|exchanges]], roteamento e protocolos como AMQP;
- **ActiveMQ**: broker com suporte a padrões e protocolos de mensageria;
- **Amazon SQS**: fila gerenciada, acessada por API, com pouca administração de servidores;
- **Redis**: pode atuar em filas simples, Streams e pub/sub, mas seu foco e suas garantias dependem do recurso usado;
- **Apache Kafka**: plataforma de streaming distribuído, voltada a eventos persistentes e alto volume, com conceitos diferentes de uma fila tradicional.

Não escolha pela popularidade do nome. Compare ordenação, retenção, confirmação, reprocessamento, throughput, latência, observabilidade e custo operacional.

## Exemplo local com Docker Compose

Para estudar um broker dedicado, pode-se usar um serviço RabbitMQ em [[Docker Compose]]:

```yaml
services:
  rabbitmq:
    image: rabbitmq:<versao-fixada>-management
    ports:
      - "127.0.0.1:5672:5672"
      - "127.0.0.1:15672:15672"
```

- `5672` é uma porta comum para comunicação AMQP;
- `15672` pode servir à interface de administração, conforme a imagem e a configuração;
- publicar em `127.0.0.1` limita o acesso à máquina local;
- substitua `<versao-fixada>` por uma versão aprovada e controlada.

Esse exemplo é para estudo. Em produção, configure credenciais, rede privada, TLS, volumes, backups, limites, monitoramento e alta disponibilidade.

## Broker na AWS

Na AWS, o [Amazon SQS](https://docs.aws.amazon.com/sqs/) é uma opção gerenciada para filas. Ele reduz a necessidade de operar um broker, mas possui modelo de consumo, retenção, visibilidade, filas FIFO e integração próprios.

Quando a aplicação precisa de compatibilidade com protocolos e APIs de brokers tradicionais, o [Amazon MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html) oferece brokers gerenciados para engines como ActiveMQ Classic e RabbitMQ. Ele se aproxima mais de operar um broker tradicional, enquanto SQS é um serviço de filas acessado por APIs AWS.

Para distribuir uma mensagem a vários destinos, o [Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) pode publicar em um tópico e encaminhar mensagens para assinantes, incluindo filas SQS.

Em uma [[VPS]], a equipe instala e administra o broker, cuidando de disco, memória, atualizações, credenciais, TLS, backups, métricas e recuperação. Na AWS, os serviços gerenciados reduzem o trabalho operacional, mas cobram por uso e introduzem regras, custos e dependência do provedor.

## Segurança

- mantenha o broker em rede privada sempre que possível;
- use autenticação e permissões mínimas por produtor e consumidor;
- proteja mensagens em trânsito com TLS e, quando necessário, em repouso;
- não coloque senhas, tokens ou dados pessoais nas mensagens sem necessidade;
- valide o formato e o tamanho da mensagem;
- não confie em um `usuarioId` recebido sem verificar autorização no backend;
- proteja a interface administrativa e não a exponha publicamente;
- registre métricas e falhas sem revelar o conteúdo sensível;
- limite conexões, tamanho de mensagem e volume por produtor.

Uma mensagem pode permanecer armazenada durante retries e retenção. Trate o broker como parte da superfície de dados do sistema e aplique as práticas de [[Segurança]].

## Boas práticas

- defina um contrato de mensagem versionado;
- inclua `eventId`, tipo, versão e horário quando forem úteis;
- faça consumidores [[Idempotência|idempotentes]];
- confirme somente depois de concluir o trabalho;
- configure retry com limite e backoff;
- use DLQ e monitore sua idade e quantidade;
- monitore tamanho das filas, latência, mensagens antigas e falhas;
- documente produtores, consumidores e responsabilidades;
- teste duplicidade, reinício, indisponibilidade, mensagens inválidas e reprocessamento;
- escolha entre fila, pub/sub e stream de acordo com a necessidade real.

**Um broker de fila é o intermediário que organiza e entrega mensagens entre componentes, ajudando a desacoplar sistemas, absorver picos e processar trabalhos de forma assíncrona.**
