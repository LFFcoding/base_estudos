# RabbitMQ

**RabbitMQ** é um *message broker* (broker de mensagens). Ele recebe mensagens de uma aplicação, organiza o caminho delas e entrega o conteúdo aos componentes que precisam processá-lo.

Ele é muito usado para executar tarefas de forma assíncrona. Por exemplo: o [[Backend]] recebe uma [[Requisição]] para criar um pedido, salva o pedido e publica uma mensagem `EnviarEmail`. Um consumidor pode enviar o e-mail depois, sem fazer o usuário esperar essa etapa.

O RabbitMQ não é apenas uma fila. Ele é o serviço que administra filas, roteamento, confirmações, consumidores, reentregas e permissões. Uma explicação mais geral desses componentes está em [[Broker de fila]], e os conceitos de roteamento estão detalhados em [[Exchanges e bindings]].

## Uma analogia

Imagine uma central de distribuição:

- o produtor (*producer*) entrega uma encomenda;
- a central (*broker*) lê o endereço e decide o caminho;
- as filas (*queues*) guardam as encomendas que aguardam transporte;
- o consumidor (*consumer*) recebe e processa a encomenda;
- a confirmação (*acknowledgement*) informa que a entrega foi concluída;
- uma área de exceções (*dead-letter queue*) guarda encomendas que falharam muitas vezes.

O RabbitMQ faz esse trabalho com mensagens. Isso permite que o produtor e o consumidor funcionem em ritmos diferentes e continuem independentes.

## Conceitos principais

- **Mensagem (*message*):** informação enviada, normalmente com um payload em [[JSON]] e metadados.
- **Produtor (*producer*):** aplicação que publica mensagens.
- **[[Exchanges e bindings|Exchange]]:** ponto que recebe a mensagem e decide para quais filas ela será encaminhada.
- **[[Exchanges e bindings|Binding]]:** regra que liga uma exchange a uma fila, possivelmente usando uma chave de roteamento.
- **Routing key:** texto usado pelo tipo da exchange para escolher o destino.
- **Fila (*queue*):** local onde a mensagem aguarda um consumidor.
- **Consumidor (*consumer*):** aplicação que recebe e processa mensagens.
- **Virtual host (*vhost*):** espaço lógico que separa exchanges, filas, permissões e configurações.
- **Acknowledgement (*ack*):** confirmação do consumidor de que terminou o processamento.
- **Publisher confirm:** confirmação do RabbitMQ de que ele aceitou a publicação.
- **Prefetch:** limite de mensagens entregues a um consumidor sem confirmação.

## Fluxo de uma mensagem

O caminho normal é:

```text
produtor -> exchange -> binding -> fila -> consumidor
```

1. o produtor publica a mensagem em uma exchange;
2. a exchange usa seu tipo e os bindings para decidir o destino;
3. a fila armazena a mensagem;
4. o consumidor recebe a mensagem;
5. o consumidor executa o trabalho;
6. o consumidor envia `ack` somente depois de concluir o trabalho;
7. o RabbitMQ pode remover a mensagem da fila depois do `ack`.

Se o consumidor cair antes do `ack`, o RabbitMQ pode entregar a mensagem novamente. Por isso, o processamento deve ser [[Idempotência|idempotente]]: repetir a mensagem não deve duplicar o efeito, como enviar duas cobranças.

## Tipos de exchange

O RabbitMQ oferece tipos diferentes de [[Exchanges e bindings|exchange]] para diferentes regras de roteamento:

- **Direct:** encaminha para bindings cuja routing key é igual à chave da mensagem.
- **Fanout:** encaminha para todas as filas ligadas à exchange, ignorando a routing key. É útil para publicar um evento para vários interessados.
- **Topic:** usa padrões na routing key, como `pedido.criado` ou `pedido.*`.
- **Headers:** escolhe o destino com base em headers da mensagem, em vez de depender principalmente da routing key.

Exemplo de topic:

```text
exchange: eventos
routing key: pedido.criado

fila de auditoria -> pedido.*
fila de notificações -> pedido.criado
```

As exchanges e suas regras fazem parte da topologia do broker. A [documentação oficial sobre exchanges](https://www.rabbitmq.com/docs/exchanges) explica como elas encaminham mensagens para filas, streams ou outras exchanges.

## Fila de trabalho e pub/sub

### Fila de trabalho

Em uma fila de trabalho, vários consumidores compartilham a mesma fila. O RabbitMQ distribui as mensagens entre eles, permitindo processar tarefas em paralelo.

Exemplo: três workers consomem a fila `emails`. Cada mensagem deve ser tratada por um worker, não por todos.

### Publicação e assinatura

No modelo *publish/subscribe*, uma mensagem pode ser encaminhada para várias filas. Cada grupo interessado possui sua própria fila e processa sua cópia.

Exemplo: quando `PedidoCriado` acontece, uma fila pode alimentar notificações, outra auditoria e outra atualização de relatórios.

Para decidir entre fila, pub/sub ou stream, consulte [[Filas]] e [[Protocolos de mensageria]].

## Confirmações e reentregas

Existem duas confirmações diferentes:

- **Consumer acknowledgement:** o consumidor informa ao RabbitMQ que terminou o processamento.
- **Publisher confirm:** o RabbitMQ informa ao produtor que aceitou a publicação.

Essas confirmações resolvem problemas diferentes. O `ack` do consumidor não prova que o produtor publicou corretamente, e o *publisher confirm* não prova que um consumidor concluiu o trabalho.

O consumidor pode:

- enviar `ack` depois do sucesso;
- enviar `nack` ou `reject` quando não conseguiu processar;
- pedir reentrega com `requeue`, quando outra tentativa fizer sentido;
- rejeitar sem reencaminhar, permitindo o uso de uma dead-letter exchange.

Não confirme antes de salvar o resultado ou concluir a ação importante. Se o processo confirmar e cair logo depois, o RabbitMQ pode remover a mensagem e o trabalho será perdido.

A [documentação oficial sobre acknowledgements e publisher confirms](https://www.rabbitmq.com/docs/confirms) explica essas garantias e seus limites.

## Retry e dead-letter queue

Nem toda falha deve ser tentada novamente. Uma falha temporária de rede pode melhorar com retry; um JSON inválido continuará falhando até ser corrigido.

Uma estratégia segura pode ser:

1. tentar processar a mensagem;
2. se a falha for temporária, reencaminhar com atraso e limite de tentativas;
3. depois do limite, enviar para uma **DLQ** (*dead-letter queue*);
4. investigar e corrigir a causa;
5. reprocessar a mensagem somente quando houver segurança.

Evite retry infinito e imediato. Ele pode criar um ciclo que consome CPU, aumenta a fila e impede o processamento de mensagens saudáveis. Use espera progressiva (*exponential backoff*), limite de tentativas e métricas.

## Durabilidade e segurança da entrega

Para reduzir o risco de perder mensagens após uma falha, normalmente é necessário combinar:

- exchange e fila duráveis;
- mensagem marcada como persistente;
- publisher confirms;
- acknowledgements manuais no consumidor;
- armazenamento e configuração de alta disponibilidade adequados;
- backups e testes de restauração.

Nenhuma opção isolada resolve todos os problemas. Uma fila durável não garante que o produtor recebeu confirmação, e um `ack` não recupera um resultado que a aplicação nunca salvou.

Também é importante entender a garantia escolhida: muitas aplicações trabalham com **at-least-once**, ou seja, tentam não perder mensagens, mas aceitam a possibilidade de duplicidade. Por isso, consumidores devem aplicar [[Idempotência]].

## Publisher confirm e mensagem sem destino

O produtor não deve presumir que escrever no socket significa que a mensagem foi aceita pelo broker. Publisher confirms permitem saber se o RabbitMQ assumiu a mensagem.

Também é necessário tratar mensagens que não encontram nenhuma fila. Dependendo da configuração, elas podem ser devolvidas ao produtor, encaminhadas para uma exchange alternativa ou descartadas. Configure o comportamento conscientemente e monitore mensagens não roteadas.

## Prefetch e capacidade do consumidor

O *prefetch* limita quantas mensagens podem ficar entregues, mas ainda sem `ack`, para um consumidor. Isso evita entregar trabalho demais a um worker lento e ajuda a controlar memória e concorrência.

Um prefetch muito baixo pode reduzir o throughput; um valor muito alto pode sobrecarregar o consumidor e dificultar a distribuição justa. O valor adequado depende do tempo de processamento, do tamanho da mensagem e da capacidade do worker.

## Exemplo local com Docker Compose

Para estudar localmente, o RabbitMQ pode ser executado com [[Docker Compose]]:

```yaml
services:
  rabbitmq:
    image: rabbitmq:<versao-fixada>-management
    ports:
      - "127.0.0.1:5672:5672"
      - "127.0.0.1:15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq

volumes:
  rabbitmq-data:
```

- `5672` é uma porta comum para clientes AMQP;
- `15672` é usada pela interface de gerenciamento da imagem com o plugin correspondente;
- `127.0.0.1` limita o acesso à máquina local;
- o volume preserva os dados do broker quando o contêiner é recriado;
- `<versao-fixada>` deve ser substituída por uma versão aprovada pela equipe.

Esse exemplo é para desenvolvimento. Em produção, configure usuário próprio, senha fora do repositório, rede privada, TLS, permissões, backups, monitoramento e alta disponibilidade. Não use credenciais padrão expostas na internet.

## Uso com Java e Quarkus

Uma aplicação [[Java]] pode publicar e consumir mensagens usando um cliente AMQP ou uma extensão de [[Quarkus]] apropriada. Um fluxo possível é:

1. o [[Backend]] recebe a requisição;
2. valida os dados e atualiza o [[PostgreSQL]];
3. publica um evento ou trabalho no RabbitMQ;
4. um worker consome a mensagem;
5. o worker executa a tarefa e envia `ack` após o sucesso.

Se a gravação no PostgreSQL e a publicação forem duas operações independentes, pode existir uma falha entre elas. O [[Outbox Pattern]] ajuda a salvar o estado principal e o evento pendente na mesma transação, para que outro processo publique a mensagem depois.

## RabbitMQ e outras opções

| Solução | Característica principal |
| --- | --- |
| RabbitMQ | broker dedicado com exchanges, bindings, filas, acknowledgements e roteamento flexível |
| [[Redis]] | armazenamento de dados que também pode oferecer listas, Streams e pub/sub; as garantias dependem do recurso usado |
| Amazon SQS | fila gerenciada acessada por API AWS, sem operar um broker AMQP tradicional |
| Apache Kafka | plataforma de streaming com tópicos, partições, offsets e retenção de eventos |

Escolha pela necessidade do sistema. RabbitMQ não é automaticamente melhor ou mais rápido; ele é adequado quando o fluxo precisa de um broker com roteamento, filas e confirmações bem definidos.

## Segurança

- use autenticação e permissões mínimas por usuário, vhost, exchange e fila;
- mantenha o broker em rede privada sempre que possível;
- proteja conexões com [[TLS]] quando a rede não for totalmente confiável;
- não coloque senhas, tokens ou dados desnecessários no payload;
- valide o formato e o tamanho das mensagens no consumidor;
- proteja a interface de gerenciamento e não a exponha publicamente;
- use filas separadas e permissões separadas quando os domínios tiverem níveis de confiança diferentes;
- não registre mensagens completas se elas contiverem dados pessoais;
- monitore conexões, filas acumuladas, mensagens antigas, redeliveries, falhas e consumidores lentos;
- teste falhas, duplicidades, reentregas, DLQs e restauração.

Esses cuidados complementam as práticas de [[Segurança]] e devem ser verificados com [[Testes]].

## RabbitMQ na AWS

O equivalente mais direto na AWS é o **Amazon MQ for RabbitMQ**, um serviço gerenciado que executa brokers RabbitMQ e reduz o trabalho de instalar, atualizar e manter a infraestrutura. Ele é indicado quando a aplicação precisa continuar usando conceitos e protocolos de um broker tradicional.

Consulte a [documentação oficial do Amazon MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html) e o guia de [uso do Amazon MQ for RabbitMQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/working-with-rabbitmq.html). Ainda é necessário configurar rede, credenciais, permissões, tamanho do broker, monitoramento, custos e estratégia de recuperação.

O **Amazon SQS** é outra opção gerenciada para filas. Ele reduz ainda mais a operação de servidores e oferece uma API própria, mas não é um RabbitMQ e não fornece a mesma topologia de exchanges, bindings e protocolos AMQP. A [documentação do Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) explica essa diferença e quando Amazon MQ, SQS ou SNS se encaixam melhor.

Em uma [[VPS]], a equipe pode instalar e operar o RabbitMQ, mas assume atualizações, armazenamento, rede, certificados, backups, alta disponibilidade, monitoramento e resposta a falhas. Na AWS, o serviço gerenciado reduz parte desse trabalho, em troca de custo, configurações próprias e dependência do provedor.

## Boas práticas

- defina um contrato de mensagem versionado;
- use identificadores únicos para rastrear eventos e detectar duplicidades;
- confirme a mensagem somente depois de concluir o processamento;
- ative publisher confirms quando a perda da publicação for relevante;
- configure retry com limite, atraso progressivo e DLQ;
- faça consumidores [[Idempotência|idempotentes]];
- escolha exchanges, routing keys e filas com nomes claros;
- ajuste prefetch conforme o comportamento real dos consumidores;
- monitore filas, latência, taxa de erro, redeliveries e mensagens não roteadas;
- teste reinício do broker, queda de consumidores, duplicidade e restauração;
- não use RabbitMQ como banco de dados principal; guarde o estado importante no [[PostgreSQL]] ou na solução adequada.

**RabbitMQ é um broker de mensagens que conecta produtores e consumidores por meio de exchanges, filas e regras de roteamento. Ele ajuda a desacoplar sistemas e executar trabalhos de forma assíncrona, mas exige configuração consciente de confirmações, reentregas, segurança, monitoramento e idempotência.**
