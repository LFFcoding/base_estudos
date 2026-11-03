# O que é Stream Processing?

**Stream Processing** (*processamento de fluxos*) é a análise contínua de dados à medida que eles chegam. Em vez de esperar que um conjunto inteiro de dados seja reunido, o sistema trata cada evento e pode atualizar resultados quase em tempo real.

Uma analogia: no processamento em lote (*batch processing*), uma loja soma as vendas no fim do dia. Com Stream Processing, cada venda passa por uma esteira e atualiza os totais e alertas enquanto a loja ainda está aberta.

## Como funciona

Um fluxo costuma ter três partes:

```text
fonte de eventos → processamento contínuo → destino dos resultados
```

- **Fonte (*source*)**: produz dados, como cliques, pagamentos, sensores ou registros de aplicação;
- **Operação (*operator*)**: filtra, transforma, agrupa, enriquece ou combina eventos;
- **Destino (*sink*)**: recebe o resultado, como um banco de dados, painel ou outro fluxo.

Exemplo conceitual: receber pagamentos, descartar tentativas recusadas, somar valores por loja em janelas de cinco minutos e enviar os totais a um painel. O fluxo pode continuar indefinidamente, por isso não é necessário esperar por um “fim” dos dados.

## Eventos, estado e tempo

Um **evento** (*event*) registra algo que aconteceu, como `PagamentoAprovado` ou `TemperaturaLida`. O processador pode tratar cada evento sozinho ou manter **estado** (*state*), isto é, informações acumuladas para calcular algo sobre eventos anteriores, como o total vendido por loja.

Como um fluxo pode nunca terminar, operações como somar ou contar precisam de um limite. Uma **janela** (*window*) agrupa eventos por uma regra, por exemplo, os pagamentos ocorridos em cada intervalo de cinco minutos.

Há duas noções de tempo importantes:

- **Tempo do evento (*event time*)**: quando o fato aconteceu, conforme registrado no próprio evento;
- **Tempo de processamento (*processing time*)**: quando o sistema conseguiu processar o evento.

Se a rede atrasar um evento, esses horários podem ser diferentes. Ferramentas como Apache Flink usam **watermarks** (*marcadores de progresso do tempo do evento*) para estimar até que ponto os dados de um intervalo já chegaram. Ainda é preciso decidir o que fazer com eventos muito atrasados.

## Stream Processing e filas

[[Filas]] normalmente guardam trabalhos para que consumidores os executem. Stream Processing lê eventos de uma fonte e calcula resultados continuamente. Um [[Broker de fila]] pode transportar e reter eventos para um processador, mas o broker e o processamento são responsabilidades distintas.

Por exemplo, [[RabbitMQ]] é frequentemente usado para distribuir tarefas. Uma plataforma de streaming, como Amazon Kinesis Data Streams ou Apache Kafka, pode manter eventos disponíveis para consumidores e reprocessamento conforme sua retenção e configuração. A tecnologia escolhida define as garantias de ordem, armazenamento e recuperação.

## Boas práticas

- defina um formato de evento versionado, com identificador e horário quando forem necessários;
- escolha conscientemente entre tempo do evento e tempo de processamento;
- estabeleça como lidar com eventos duplicados, fora de ordem ou atrasados;
- limite o estado mantido e configure como ele será recuperado após uma falha;
- torne efeitos externos idempotentes, pois um evento pode ser processado novamente;
- monitore atraso do fluxo, volume, falhas, eventos atrasados e capacidade de processamento;
- proteja dados pessoais e credenciais, e valide os eventos recebidos;
- use processamento em lote quando a resposta imediata não trouxer benefício, pois sistemas contínuos exigem operação e monitoramento constantes.

## Equivalente na AWS

Não há um único serviço que corresponda a todo o conceito. **Amazon Kinesis Data Streams** pode receber, armazenar e disponibilizar registros de eventos; **Amazon Managed Service for Apache Flink** executa aplicações Flink que leem fontes de streaming, transformam os dados e escrevem resultados em destinos. O serviço gerenciado opera parte da infraestrutura do Flink, enquanto uma instalação própria exige que a equipe cuide do cluster e de sua disponibilidade.

## Relação com outros assuntos

Stream Processing se conecta a [[Protocolos de mensageria]], [[Broker de fila]] e [[Redis]], que podem fazer parte do transporte ou armazenamento de eventos. A confiabilidade dos eventos publicados pode envolver o [[Outbox Pattern]]. Use [[Observability]] para acompanhar atraso e falhas e [[Testes]] para validar eventos repetidos, fora de ordem e atrasados.

## Referências

- [Conceitos de processamento de fluxos e tempo no Apache Flink](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/)
- [O que é Amazon Kinesis Data Streams?](https://docs.aws.amazon.com/streams/latest/dev/)
- [O que é Amazon Managed Service for Apache Flink?](https://docs.aws.amazon.com/managed-flink/latest/java/)
