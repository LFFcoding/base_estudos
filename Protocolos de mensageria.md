# Protocolos de mensageria

**Protocolos de mensageria** são conjuntos de regras que definem como sistemas trocam mensagens. Eles descrevem o formato da comunicação, como uma mensagem é enviada, como o destino é indicado, como o recebimento é confirmado e como erros podem ser tratados.

Uma mensagem pode representar um trabalho, como `EnviarEmail`, ou um acontecimento, como `PedidoCriado`. O protocolo define como essa mensagem atravessa a rede entre o produtor, o [[Broker de fila]] e o consumidor, dentro de um fluxo de [[Filas]].

## Uma analogia

Imagine que empresas de transporte troquem encomendas:

- a encomenda é a mensagem;
- o endereço é o destino;
- a etiqueta informa o tipo e o conteúdo;
- a transportadora é o broker;
- o protocolo define o formato da etiqueta, a entrega, a confirmação e o tratamento de extravios.

Sem regras compartilhadas, cada sistema teria que inventar uma forma diferente de conversar.

## Protocolo, broker e fila

Esses conceitos se relacionam, mas não são iguais:

- **protocolo**: regras e formato usados na comunicação;
- **broker**: serviço que recebe, armazena, roteia e entrega mensagens;
- **fila**: lugar onde mensagens aguardam processamento;
- **cliente**: aplicação que publica ou consome mensagens;
- **mensagem**: conteúdo e metadados enviados;
- **API**: interface que permite usar uma capacidade de um sistema.

RabbitMQ é um broker que pode conversar usando AMQP. AMQP é o protocolo. Uma fila criada dentro do broker é o local onde algumas mensagens aguardam consumidores.

## O que um protocolo pode definir?

Um protocolo de mensageria pode definir:

- como abrir e encerrar uma conexão;
- como autenticar um cliente;
- como representar uma mensagem;
- como identificar o destino;
- como informar headers e metadados;
- como confirmar o processamento (*acknowledgement*);
- como rejeitar ou reenviar uma mensagem;
- como preservar ordem;
- como tratar mensagens duplicadas;
- como negociar tamanho, versão e recursos;
- como proteger a conexão com [[TLS]].

Nem todo protocolo oferece todas essas garantias. A aplicação precisa ler a documentação do broker e do cliente escolhido.

## AMQP

**AMQP** (*Advanced Message Queuing Protocol*) é um protocolo voltado a mensageria, com conceitos de produtores, consumidores, filas, roteamento e confirmações.

Em uma implementação como RabbitMQ, um fluxo pode ser:

```text
produtor -> exchange -> binding -> fila -> consumidor
```

- o produtor publica a mensagem;
- a [[Exchanges e bindings|exchange]] recebe a mensagem;
- o [[Exchanges e bindings|binding]] define a regra de roteamento;
- a fila armazena a mensagem;
- o consumidor recebe e confirma o processamento.

AMQP é útil quando a aplicação precisa de roteamento explícito, confirmações, filas e controle sobre o fluxo. RabbitMQ é uma implementação conhecida, mas a existência do protocolo não significa que todos os brokers ofereçam exatamente os mesmos recursos.

## MQTT

**MQTT** (*Message Queuing Telemetry Transport*) é um protocolo leve de publicação e assinatura (*publish/subscribe*), muito usado em dispositivos conectados e Internet das Coisas (*IoT*).

Os clientes publicam em tópicos e outros clientes assinam esses tópicos:

```text
dispositivo -> casa/sala/temperatura -> broker MQTT -> painel
```

Exemplo conceitual:

```text
PUBLISH casa/sala/temperatura
23.5

SUBSCRIBE casa/sala/temperatura
```

O MQTT possui níveis de qualidade de serviço (*QoS*) que equilibram confirmação, custo e possibilidade de duplicidade. Um dispositivo com conexão instável pode usar MQTT para trocar mensagens pequenas sem manter uma conexão pesada com cada consumidor.

## STOMP

**STOMP** (*Simple Text Oriented Messaging Protocol*) é um protocolo textual e simples. Ele usa frames, parecidos com pequenos envelopes de texto, e pode ser usado sobre conexões como WebSocket ou TCP.

Um frame conceitual de envio é:

```text
SEND
destination:/queue/emails
content-type:application/json

{"usuarioId":42,"tipo":"boas-vindas"}
^@
```

O formato fácil de ler ajuda em integrações simples e aplicações que precisam enviar atualizações para interfaces conectadas. O broker ainda precisa definir como filas, confirmações, retries e permissões funcionam.

## JMS não é exatamente um protocolo

**JMS** (*Java Message Service*) é uma API e um conjunto de interfaces para aplicações Java trabalharem com mensageria. Ele permite que o código use conceitos como `Queue`, `Topic`, produtor e consumidor sem ficar preso a todos os detalhes de uma implementação.

JMS é diferente de AMQP:

- JMS é uma API usada pelo código Java;
- AMQP é um protocolo de comunicação;
- um provedor JMS pode usar um protocolo ou transporte específico por baixo;
- trocar o broker pode exigir ajustes conforme os recursos utilizados.

No [[Java]] e no [[Quarkus]], a equipe deve verificar qual extensão ou cliente será usado e quais garantias o broker oferece.

## Kafka e streaming

O Apache Kafka possui APIs e protocolos próprios para publicar e consumir registros em tópicos particionados. Ele é frequentemente usado como plataforma de **event streaming**, com retenção de eventos, partições, offsets e grupos de consumidores.

Kafka e uma fila tradicional podem resolver problemas parecidos, mas o modelo mental é diferente:

- numa fila, o consumidor normalmente remove ou confirma o trabalho;
- em Kafka, o registro permanece conforme a política de retenção e o consumidor controla sua posição (*offset*);
- Kafka é forte em alto volume, histórico e reprocessamento;
- um broker tradicional pode ser mais simples para tarefas assíncronas individuais.

Escolha pela necessidade de retenção, ordem, throughput, reprocessamento e operação, não apenas pelo nome da tecnologia.

## HTTP também é um protocolo de comunicação

HTTP é muito usado no modelo requisição e resposta:

```text
cliente -> requisição HTTP -> servidor
cliente <- resposta HTTP  <- servidor
```

Protocolos de mensageria costumam ser escolhidos quando:

- o produtor não deve esperar o consumidor terminar;
- mensagens precisam aguardar quando o consumidor está indisponível;
- vários consumidores precisam processar trabalhos;
- o sistema precisa de retries, confirmação ou reprocessamento;
- o fluxo é orientado a eventos.

HTTP continua sendo útil para comunicação direta, consultas e comandos que precisam de resposta imediata. Um sistema pode usar HTTP entre o [[Frontend]] e o [[Backend]] e mensageria entre serviços internos.

## Entrega, ordem e duplicidade

O protocolo e o broker podem oferecer diferentes garantias:

- **at-most-once**: no máximo uma entrega, com possibilidade de perda;
- **at-least-once**: pelo menos uma entrega, com possibilidade de duplicidade;
- **FIFO**: preservação de ordem em um escopo definido;
- **ack**: confirmação de processamento;
- **redelivery**: entrega novamente após falha ou timeout;
- **retention**: tempo que a mensagem permanece armazenada.

Mesmo quando existe ack, uma falha entre terminar o trabalho e confirmar a mensagem pode provocar redelivery. Por isso, consumidores devem ser [[Idempotência|idempotentes]] e usar identificadores únicos.

## Segurança

- use autenticação para produtores, consumidores e administração;
- dê permissões mínimas por fila, tópico ou operação;
- proteja o tráfego com [[TLS]] quando a rede não for totalmente confiável;
- considere criptografia em repouso para mensagens armazenadas;
- não coloque senhas, tokens ou chaves privadas no payload;
- valide tamanho, tipo e formato da mensagem;
- limite conexões e volume por cliente;
- proteja interfaces administrativas;
- registre falhas e métricas sem expor dados sensíveis;
- trate mensagens como dados que fazem parte da superfície de [[Segurança]].

## Como escolher um protocolo

Pergunte:

1. o fluxo precisa de fila, pub/sub ou stream?
2. os clientes estão em dispositivos limitados ou em servidores?
3. é necessária compatibilidade com JMS, AMQP ou outro protocolo?
4. mensagens podem ser duplicadas?
5. a ordem precisa ser global ou somente por entidade?
6. quanto tempo os eventos precisam ficar disponíveis?
7. quem cuidará do broker e do monitoramento?
8. qual latência e volume são esperados?
9. quais dados precisam de criptografia e controle de acesso?

## Protocolos e AWS

Na AWS, o [Amazon SQS](https://docs.aws.amazon.com/sqs/) oferece uma fila gerenciada acessada por APIs AWS, SDKs e chamadas HTTP. Ele não é uma implementação AMQP tradicional, por isso a aplicação deve usar o modelo de visibilidade, retenção, DLQ e confirmação do próprio SQS.

O [Amazon MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/welcome.html) é mais próximo de operar um broker tradicional e oferece engines como ActiveMQ Classic e RabbitMQ. Ele é útil quando a aplicação precisa de compatibilidade com protocolos, APIs e padrões já usados.

O [Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) trabalha com tópicos e assinantes, podendo encaminhar mensagens para filas SQS e outros destinos.

Em uma [[VPS]], a equipe instala o broker e administra protocolos, portas, certificados, usuários, filas, armazenamento, atualizações e monitoramento. Os serviços AWS reduzem a operação de servidores, mas impõem APIs, limites, custos e dependência do provedor.

## Boas práticas

- escolha um protocolo que os clientes suportem bem;
- defina um contrato de mensagem versionado;
- documente destinos, headers, payloads e garantias;
- configure ack, retry, timeout, retenção e DLQ conscientemente;
- faça consumidores idempotentes;
- monitore mensagens antigas, falhas, latência e tamanho das filas;
- teste mensagens duplicadas, fora de ordem, inválidas e grandes demais;
- mantenha bibliotecas e clientes atualizados;
- não misture detalhes do broker com regras de negócio sem necessidade;
- proteja credenciais e conexões usando as práticas de [[Segurança]].

**Protocolos de mensageria são as regras que permitem que sistemas troquem mensagens de forma compreensível, controlada e segura.** O protocolo define a comunicação; o broker opera a entrega; a fila organiza o trabalho.
