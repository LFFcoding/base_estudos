# Filas

Em sistemas, uma **fila** (*queue*) é uma estrutura que guarda trabalhos ou mensagens para serem processados depois. Normalmente, quem coloca a mensagem na fila é chamado de **produtor** (*producer*) e quem a retira para executar o trabalho é chamado de **consumidor** (*consumer*) ou *worker*.

## Uma analogia

Imagine uma padaria com uma fila de senhas:

- a pessoa recebe uma senha e entra na fila;
- o atendente chama uma senha por vez;
- se muitas pessoas chegarem, elas esperam sem bloquear o atendente;
- se o atendente parar por alguns minutos, as senhas continuam aguardando.

Uma fila de software faz algo parecido. Ela separa quem cria o trabalho de quem executa o trabalho.

## Por que usar uma fila?

Sem uma fila, o [[Backend]] pode fazer tudo dentro da mesma [[Requisição]]:

1. receber o pedido;
2. gerar um relatório;
3. enviar um e-mail;
4. processar uma imagem;
5. responder somente depois de tudo terminar.

Se uma dessas tarefas demora ou falha, a pessoa espera mais e a requisição pode expirar. Com uma fila, o backend registra o trabalho, responde rapidamente e um worker o processa em segundo plano.

## Exemplo de fluxo

Um cadastro pode funcionar assim:

1. o [[Frontend]] envia uma [[Requisição]] para o [[Backend]];
2. o backend valida e salva o cadastro;
3. o backend coloca uma mensagem `enviar-boas-vindas` na fila;
4. o backend responde que o cadastro foi criado;
5. um worker retira a mensagem;
6. o worker envia o e-mail;
7. o worker confirma que a mensagem foi processada.

A tarefa secundária não precisa bloquear a resposta principal.

## Conceitos importantes

- **Mensagem (*message*)**: dados que descrevem o trabalho, normalmente em [[JSON]].
- **Produtor (*producer*)**: componente que publica uma mensagem.
- **Consumidor (*consumer*)**: componente que recebe e processa a mensagem.
- **Worker**: processo que executa o trabalho recebido.
- **[[Broker de fila|Broker]]**: serviço que armazena e distribui mensagens.
- **Acknowledgement (*ack*)**: confirmação de que a mensagem foi processada.
- **Retry**: nova tentativa depois de uma falha.
- **Dead-letter queue (DLQ)**: fila para mensagens que falharam várias vezes.
- **Visibilidade (*visibility timeout*)**: período em que uma mensagem recebida fica escondida de outros consumidores enquanto é processada.

## Ordem e entrega

Nem toda fila garante exatamente a mesma ordem ou uma única entrega.

- **FIFO (*First In, First Out*)**: tenta processar na ordem de chegada;
- **at-most-once**: uma mensagem pode ser perdida, mas não deve ser repetida;
- **at-least-once**: uma mensagem não deveria ser perdida, mas pode ser entregue mais de uma vez;
- **exactly-once**: promete uma única entrega em condições específicas, mas não elimina todos os problemas da operação completa.

Em muitos sistemas distribuídos, o consumidor deve estar preparado para receber a mesma mensagem duas vezes. Por isso, o processamento deve ser [[Idempotência|idempotente]]: repetir a operação não pode causar um efeito incorreto, como cobrar duas vezes o mesmo pagamento.

## Exemplo simples com Redis

O [[Redis]] pode representar uma fila simples usando uma lista:

```text
LPUSH emails "{\"usuarioId\":42,\"tipo\":\"boas-vindas\"}"
BRPOP emails 30
```

- `LPUSH` coloca uma mensagem na lista;
- `BRPOP` espera até aparecer uma mensagem e a retira;
- `30` é o tempo máximo de espera em segundos.

Esse exemplo é útil para estudo, mas uma fila de produção precisa tratar confirmação, mensagens repetidas, falhas, observabilidade e recuperação. O Redis também possui Streams, que oferecem recursos mais adequados para alguns fluxos de eventos.

## Filas e pub/sub são diferentes

Em uma fila, normalmente uma mensagem é entregue a um consumidor para que um trabalho seja realizado. Em **pub/sub**, uma mensagem publicada em um canal pode ser recebida por vários assinantes conectados.

- fila: “alguém precisa fazer este trabalho”;
- pub/sub: “avise todos os interessados sobre este acontecimento”.

Se um assinante de pub/sub estiver desconectado, ele pode perder a mensagem. Para processamento confiável, use uma fila ou um mecanismo com armazenamento e confirmação.

## Filas e banco de dados

Uma fila não substitui automaticamente o [[PostgreSQL]]. O PostgreSQL pode guardar o estado principal de um pedido, enquanto a fila coordena tarefas que precisam acontecer depois.

Um padrão comum é salvar o pedido e o evento de envio na mesma transação, usando o [[Outbox Pattern]]. Depois, um publicador lê os eventos pendentes e os envia para a fila. Isso reduz o risco de salvar o pedido e perder a mensagem entre duas operações separadas.

## Filas na AWS

Na AWS, o [Amazon SQS](https://docs.aws.amazon.com/sqs/) é um serviço gerenciado de filas. Ele ajuda a desacoplar componentes e permite que produtores e consumidores trabalhem em ritmos diferentes.

O SQS reduz a necessidade de instalar e manter um broker em uma [[VPS]], mas envolve cobrança por uso, configuração de permissões IAM, filas padrão ou FIFO, tempo de visibilidade, retenção e tratamento de mensagens mortas. Para publicar uma mensagem a vários destinos, o Amazon SNS pode ser combinado com filas SQS.

Em uma [[VPS]], a equipe pode executar um [[Broker de fila|broker]] com [[Docker Compose]], mas precisa cuidar de atualizações, armazenamento, backups, monitoramento, segurança e alta disponibilidade.

## Segurança

- não coloque senhas ou tokens diretamente nas mensagens;
- proteja o acesso ao broker com autenticação, autorização e rede privada;
- use criptografia quando mensagens atravessarem redes não confiáveis;
- limite quais produtores podem publicar e quais consumidores podem ler;
- valide o conteúdo das mensagens antes de processá-las;
- não confie em um `usuarioId` sem verificar a autorização no backend;
- defina retenção e políticas para mensagens que falham;
- masque dados sensíveis nos logs.

Uma fila pode conter dados importantes por mais tempo do que uma requisição. Trate suas mensagens como dados que precisam das práticas de [[Segurança]].

## Boas práticas

- mantenha as mensagens pequenas e com um formato versionado;
- inclua um identificador único para facilitar deduplicação e rastreamento;
- faça consumidores [[Idempotência|idempotentes]];
- defina timeout, retry com espera progressiva (*exponential backoff*) e limite de tentativas;
- envie mensagens problemáticas para uma DLQ;
- monitore tamanho da fila, idade da mensagem, falhas e tempo de processamento;
- escale consumidores quando o volume aumentar;
- evite colocar tarefas que precisam de resposta imediata em uma fila sem informar o novo comportamento à pessoa;
- documente quem produz, quem consome e qual resultado é esperado;
- teste reinício do worker, mensagens duplicadas, falhas e recuperação.

**Uma fila permite separar a produção de um trabalho do seu processamento, melhorando desacoplamento, resiliência e capacidade de lidar com picos.**
