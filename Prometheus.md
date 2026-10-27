# O que é Prometheus?

**Prometheus** é um sistema de código aberto para monitorar métricas numéricas ao longo do tempo. Ele guarda os valores como **séries temporais** (*sequências de medições com data e hora*) e oferece a linguagem PromQL para consultá-los.

Uma analogia: sensores registram a temperatura de uma sala em vários horários; o Prometheus armazena essas medições para que seja possível consultar o valor atual, observar tendências e detectar mudanças.

## Como funciona

O Prometheus normalmente consulta os alvos em intervalos regulares, buscando suas métricas em um endpoint HTTP. Esse modelo é chamado de *pull*. Aplicações podem expor métricas diretamente ou por meio de exporters, que traduzem dados de outros sistemas.

```promql
rate(http_requests_total[5m])
```

Essa consulta PromQL estima quantas requisições por segundo ocorreram, usando as medições dos últimos cinco minutos. Prometheus também pode avaliar regras e encaminhar alertas a um Alertmanager.

Prometheus é voltado principalmente a métricas; não substitui sozinho sistemas de armazenamento e consulta de logs ou traces.

## Boas práticas

- defina nomes e unidades claros para as métricas;
- use labels com valores limitados e evite IDs únicos, que podem criar muitas séries;
- configure retenção e capacidade de armazenamento conforme volume e necessidade;
- monitore a própria coleta e confirme se os alvos continuam acessíveis;
- construa alertas que indiquem impacto ou ação necessária, em vez de alertar sobre qualquer variação.

## Equivalente na AWS

O **Amazon Managed Service for Prometheus** é um serviço gerenciado e compatível com Prometheus, que reduz o trabalho de operar a camada de métricas em escala. Ele pode receber métricas de clusters EKS e trabalhar junto com [[Grafana]]; a configuração e os limites do serviço gerenciado diferem de uma instalação autogerenciada.

## Relação com outros assuntos

Prometheus pertence à área [[Observability]]. Pode receber métricas produzidas por aplicações instrumentadas com [[OpenTelemetry]] e fornecer dados para painéis do [[Grafana]].

## Referências

- [Visão geral do Prometheus](https://prometheus.io/docs/introduction/overview/)
- [Amazon Managed Service for Prometheus](https://docs.aws.amazon.com/prometheus/latest/userguide/what-is-Amazon-Managed-Service-Prometheus.html)
