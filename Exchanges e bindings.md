# Exchanges e bindings

Em sistemas de [[Protocolos de mensageria|mensageria]], **exchange** e **binding** são elementos usados para decidir para onde uma mensagem deve ir. Eles aparecem com destaque no [[RabbitMQ]], mas a ideia geral de separar publicação e destino também existe em outros brokers.

## Resposta curta

- **Exchange:** recebe a mensagem do produtor e aplica uma regra de roteamento.
- **Binding:** regra que liga uma exchange a uma fila, stream ou outra exchange.
- **Fila (*queue*):** lugar onde a mensagem espera até ser entregue ao consumidor.
- **Routing key:** informação da mensagem que pode ser usada pela exchange para escolher os destinos.

O caminho costuma ser:

```text
produtor -> exchange -> binding -> fila -> consumidor
```

A exchange normalmente não é o lugar onde a mensagem fica aguardando processamento. Ela funciona como uma central de distribuição; a fila é o local que guarda a mensagem para um consumidor.

## Uma analogia

Imagine um centro de distribuição:

- o produtor entrega uma encomenda;
- a exchange é o setor que lê as informações da encomenda;
- o binding é a regra que diz qual rota atende determinado tipo de encomenda;
- a fila é o caminhão ou depósito do destino;
- o consumidor é quem recebe e processa a encomenda.

Uma mesma mensagem pode ser encaminhada para várias filas quando existem vários bindings compatíveis. Isso permite que auditoria, notificações e relatórios recebam cópias independentes do mesmo evento.

## Exchange

Uma exchange é um ponto de entrada para mensagens publicadas. O produtor envia a mensagem para a exchange, e não precisa conhecer diretamente cada fila que será usada.

Essa separação reduz o acoplamento:

- o produtor conhece o nome da exchange e a routing key;
- a configuração da exchange conhece os bindings;
- os consumidores conhecem suas filas;
- as filas podem mudar de destino sem que o produtor precise ser reescrito.

Uma exchange pode encaminhar mensagens para filas, streams ou outras exchanges, conforme a topologia configurada. Ela pode existir sem nenhum binding, mas nesse caso não haverá uma fila compatível para receber a mensagem.

## Binding

Um binding é uma relação configurada entre uma exchange e um destino. O destino costuma ser uma fila, mas pode ser outra exchange.

Um binding pode conter:

- exchange de origem;
- fila ou exchange de destino;
- binding key (chave ou padrão usado na regra);
- argumentos adicionais, quando o tipo de exchange precisar deles.

O binding não é a mensagem e não é o consumidor. Ele é uma regra permanente da topologia, até ser alterado ou removido.

Exemplo:

```text
exchange: eventos
fila: auditoria
binding key: pedido.*
```

Nesse exemplo, a fila `auditoria` demonstra interesse em mensagens da exchange `eventos` cuja chave siga o padrão `pedido.*`, se a exchange for do tipo `topic`.

## Routing key e binding key

Esses nomes podem causar confusão:

- **routing key:** chave enviada pelo produtor junto com a mensagem;
- **binding key:** chave ou padrão definido no binding;
- a exchange compara as duas conforme seu tipo.

Em algumas documentações, a chave do binding também é chamada de *routing key*. Para estudar com clareza, podemos pensar nela como a regra de comparação:

```text
routing key da mensagem: pedido.criado
binding key da fila:     pedido.*
resultado:               corresponde
```

No tipo `direct`, a comparação costuma ser exata. No tipo `topic`, a regra pode usar curingas. No tipo `fanout`, a routing key é ignorada.

## Tipos comuns de exchange

### Direct

Encaminha a mensagem quando a routing key é exatamente igual à binding key.

```text
exchange: tarefas
routing key: email

fila emails      -> binding key: email
fila pagamentos  -> binding key: pagamento
```

Uma mensagem com a chave `email` vai para `emails`, mas não para `pagamentos`.

Use `direct` quando os destinos forem explícitos e simples, como `email`, `pagamento` ou `relatorio`.

### Fanout

Encaminha uma cópia para todas as filas ligadas à exchange. A routing key é ignorada.

```text
exchange: pedido-criado
    |-> fila notificacoes
    |-> fila auditoria
    |-> fila relatorios
```

É adequado para publicar um evento para vários interessados independentes.

### Topic

Compara a routing key com padrões separados por pontos. Em RabbitMQ:

- `*` representa exatamente uma parte;
- `#` representa zero ou mais partes.

Exemplo:

```text
routing key da mensagem: pedido.criado

fila geral    -> pedido.#
fila pedidos  -> pedido.*
fila usuarios -> usuario.#
```

As filas `geral` e `pedidos` recebem `pedido.criado`; `usuarios` não recebe.

`topic` é útil quando a aplicação possui categorias hierárquicas, como `pedido.criado`, `pedido.cancelado` e `usuario.cadastrado`.

### Headers

Usa headers da mensagem para escolher o destino, em vez de depender principalmente da routing key. Pode ser útil quando a regra depende de várias propriedades, mas aumenta a complexidade da configuração.

## Exemplo completo de roteamento

Suponha que uma aplicação publique o evento:

```json
{
  "tipo": "PedidoCriado",
  "pedidoId": 42,
  "usuarioId": 7
}
```

Com a exchange `eventos` do tipo `topic`:

```text
fila auditoria       -> pedido.#
fila notificacoes     -> pedido.criado
fila somente-pedidos  -> pedido.*
fila usuarios         -> usuario.#
```

Se a mensagem for publicada com a routing key `pedido.criado`, ela será encaminhada para as três primeiras filas. A fila `usuarios` não receberá a mensagem.

Cada consumidor pode processar sua própria cópia. Se o envio de notificação falhar, isso não precisa impedir a auditoria, desde que cada fila e consumidor tenha seu próprio controle de falha.

## Exchange-to-exchange

RabbitMQ também permite ligar uma exchange a outra exchange. Isso cria uma topologia em camadas:

```text
exchange geral -> exchange pedidos -> fila pedidos
                              \-> fila auditoria
```

Esse recurso pode ajudar a separar regras de roteamento por domínio. Use-o com moderação: uma topologia muito complexa dificulta descobrir por onde a mensagem passou.

## Exchange padrão

O RabbitMQ possui uma exchange padrão, cujo nome é uma string vazia. Quando uma fila é criada, ela é ligada automaticamente à exchange padrão usando o próprio nome da fila como routing key.

Isso permite publicar diretamente pelo nome da fila, mas ainda existe uma exchange por trás do fluxo. Para topologias explícitas, prefira criar exchanges nomeadas e documentar seus bindings.

## O que acontece quando não há binding?

Se nenhuma regra combinar com a mensagem, ela não será entregue a uma fila. Dependendo da configuração do produtor e do broker, ela pode:

- ser devolvida ao produtor;
- ser enviada para uma exchange alternativa;
- ser descartada.

Para mensagens importantes:

- use publisher confirms;
- trate mensagens não roteadas;
- configure uma estratégia para destinos desconhecidos;
- monitore contadores e logs de mensagens sem destino.

Não presuma que publicar em uma exchange significa que algum consumidor recebeu a mensagem.

## Configuração em etapas

Uma topologia comum é criada assim:

1. declarar uma exchange, normalmente durável em produção;
2. declarar uma fila, com nome e propriedades adequados;
3. criar um binding entre a exchange e a fila;
4. publicar mensagens com routing keys documentadas;
5. iniciar o consumidor da fila;
6. monitorar mensagens acumuladas, falhas e destinos não roteados.

Em uma aplicação com [[RabbitMQ]], a topologia pode ser declarada pelo código, por ferramentas de administração ou por infraestrutura automatizada. Todos os ambientes devem usar uma configuração conhecida e revisável.

## Exemplo local com Docker Compose

Para estudar a topologia localmente, use RabbitMQ com a interface de gerenciamento em [[Docker Compose]]:

```yaml
services:
  rabbitmq:
    image: rabbitmq:<versao-fixada>-management
    ports:
      - "127.0.0.1:5672:5672"
      - "127.0.0.1:15672:15672"
```

- `5672` é uma porta comum de comunicação AMQP;
- `15672` é usada pela interface de gerenciamento da imagem com o plugin correspondente;
- `127.0.0.1` limita o acesso à máquina local;
- substitua `<versao-fixada>` por uma versão aprovada, em vez de usar `latest`.

O exemplo inicia o broker, mas ainda é necessário criar a exchange, a fila e o binding. Em produção, configure volume, autenticação, rede privada, TLS, backups e monitoramento.

## Exchange, binding e consumidor

Esses elementos têm responsabilidades diferentes:

| Elemento | Responsabilidade |
| --- | --- |
| Exchange | decidir os destinos possíveis |
| Binding | declarar a regra que liga origem e destino |
| Fila | armazenar mensagens aguardando processamento |
| Consumidor | executar o trabalho e confirmar o resultado |
| Routing key | informar a categoria ou destino pretendido da mensagem |

Uma exchange e um binding não substituem a lógica do consumidor. O consumidor ainda precisa validar o payload, tratar erros, evitar duplicidade com [[Idempotência]] e enviar `ack` somente após o sucesso.

## Exchange, binding e outbox

Quando o [[Backend]] salva uma mudança no [[PostgreSQL]] e publica um evento em operações separadas, pode ocorrer uma falha entre as duas etapas. O [[Outbox Pattern]] registra a mudança e o evento na mesma transação; depois, um publicador envia o evento para a exchange.

Esse padrão não elimina a necessidade de bindings. Ele torna a publicação mais confiável, enquanto os bindings continuam decidindo quais filas receberão o evento.

## Boas práticas

- use nomes claros e consistentes para exchanges, filas e routing keys;
- documente o tipo de cada exchange e o significado de cada padrão;
- prefira exchanges duráveis para fluxos importantes;
- evite bindings genéricos demais, como `#`, sem uma razão clara;
- trate mensagens sem destino com publisher confirms, retorno ou exchange alternativa;
- use uma fila separada para cada grupo de consumidores com responsabilidade diferente;
- mantenha consumidores idempotentes, pois uma mensagem pode ser entregue novamente;
- evite topologias profundas e difíceis de investigar;
- monitore filas, mensagens não roteadas e alterações de bindings;
- não coloque segredos ou dados desnecessários na routing key;
- teste roteamento correto, mensagem sem correspondência, duplicidade e falha do consumidor.

## AWS

O **Amazon MQ for RabbitMQ** é a alternativa gerenciada mais próxima quando a aplicação precisa de RabbitMQ, exchanges, bindings e protocolos de broker tradicionais. A AWS administra parte da infraestrutura, mas a equipe continua responsável por desenhar a topologia, configurar permissões, monitorar filas e controlar custos. Consulte a [documentação oficial do Amazon MQ](https://docs.aws.amazon.com/amazon-mq/latest/developer-guide/working-with-rabbitmq.html).

O **Amazon SQS** oferece filas gerenciadas, mas não possui a mesma topologia de exchanges e bindings do RabbitMQ. Quando é necessário distribuir uma mensagem para várias filas, o **Amazon SNS** pode publicar em um tópico e encaminhar para assinantes, incluindo filas SQS. Esse modelo é parecido com um fanout, mas possui APIs, garantias e configurações próprias da AWS. Veja a [documentação oficial que compara SQS, SNS e Amazon MQ](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html).

Em uma [[VPS]], a equipe pode instalar RabbitMQ e configurar livremente exchanges e bindings, mas precisa cuidar de atualizações, armazenamento, segurança, backups, alta disponibilidade e recuperação.

**Exchange é o roteador; binding é a regra que conecta o roteador a um destino. Juntos, eles permitem publicar uma mensagem sem acoplar o produtor diretamente a uma fila específica.**
