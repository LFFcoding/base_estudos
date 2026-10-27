# Área: Observability

**Observability** (*observabilidade*) é a capacidade de entender como o sistema está funcionando a partir dos sinais que ele produz, como métricas, logs e traces.

Uma analogia: é o painel de um carro. Ele não dirige por você, mas mostra velocidade, alertas e informações que ajudam a descobrir quando algo precisa de atenção.

## Conceitos desta área

- [[OpenTelemetry]]: padrões, APIs e componentes para instrumentar e encaminhar telemetria;
- [[Prometheus]]: coleta e consulta métricas como séries temporais;
- [[Grafana]]: consulta fontes de dados e apresenta dashboards e alertas.

As ferramentas podem trabalhar juntas: a aplicação emite sinais, coletores os encaminham, sistemas de armazenamento os mantêm e dashboards ajudam a interpretá-los.

## Boas práticas

- observe sintomas que importam para usuários, além da saúde das máquinas;
- use métricas, logs e traces em conjunto quando necessário;
- evite colocar identificadores ilimitados ou dados pessoais como labels;
- proteja logs, dashboards e alertas como parte da superfície de segurança.

## Equivalentes na AWS

**AWS Distro for OpenTelemetry (ADOT)** pode coletar e encaminhar telemetria; **Amazon Managed Service for Prometheus** oferece armazenamento e consulta compatíveis com Prometheus; **Amazon Managed Grafana** fornece dashboards gerenciados. CloudWatch e X-Ray também cobrem partes de métricas, logs e traces, mas a combinação exata depende dos dados e do desenho operacional.

## Referências

- [Conceitos de OpenTelemetry](https://opentelemetry.io/docs/concepts/)
- [Visão geral do Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Monitorar Prometheus no EKS](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html)
