# O que é OpenTelemetry?

**OpenTelemetry**, muitas vezes abreviado como **OTel**, é um conjunto aberto de padrões, APIs, bibliotecas e ferramentas para instrumentar aplicações e produzir telemetria: **métricas**, **logs** e **traces** (*rastros de uma operação entre componentes*).

Uma analogia: sensores registram o que acontece em uma fábrica, e um sistema de transporte leva esses registros até as equipes que os analisam. OpenTelemetry padroniza parte dos sensores e do transporte; não é, por si só, o lugar onde todos os dados ficam guardados ou são exibidos.

## Como funciona

Uma aplicação instrumentada pode enviar dados a um **OpenTelemetry Collector**, que recebe, processa e encaminha sinais para sistemas de armazenamento e análise.

```text
Aplicação → OpenTelemetry Collector → backend de telemetria → dashboard ou alerta
```

O protocolo OTLP (*OpenTelemetry Protocol*) é um formato comum de transporte. Bibliotecas e agentes ajudam a coletar dados, e os destinos podem ser diferentes conforme o tipo de sinal e a arquitetura.

## Boas práticas

- prefira instrumentação automática quando ela atender ao caso e adicione instrumentação manual para operações importantes;
- use nomes e atributos consistentes para facilitar consultas entre serviços;
- evite incluir senhas, tokens ou dados pessoais na telemetria;
- controle atributos com muitos valores diferentes, como IDs únicos, para não explodir o número de séries de métricas;
- dimensione, proteja e monitore o Collector como parte da infraestrutura.

## Equivalente na AWS

O **AWS Distro for OpenTelemetry (ADOT)** é uma distribuição da AWS compatível com OpenTelemetry, com componentes para coletar e encaminhar telemetria a destinos como CloudWatch e X-Ray. É uma opção compatível do ecossistema, não um backend único que substitui todas as ferramentas de observabilidade.

## Relação com outros assuntos

OpenTelemetry pertence à área [[Observability]]. Pode encaminhar métricas para [[Prometheus]] e dados para outras plataformas; [[Grafana]] pode consultar fontes compatíveis e apresentá-las em dashboards.

## Referências

- [Conceitos de OpenTelemetry](https://opentelemetry.io/docs/concepts/)
- [AWS Distro for OpenTelemetry](https://aws-otel.github.io/docs/introduction)
