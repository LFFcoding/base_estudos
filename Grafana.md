# O que é Grafana?

**Grafana** é uma plataforma para consultar, visualizar e acompanhar dados de diferentes fontes. Com ela, equipes montam **dashboards** (*painéis*) com gráficos e podem criar alertas a partir dos dados consultados.

Uma analogia: se os sistemas de monitoramento são instrumentos que medem partes de uma máquina, Grafana é o painel que reúne essas leituras para que uma pessoa entenda o estado geral.

Grafana não coleta nem armazena automaticamente todos os dados. É conectada a fontes como [[Prometheus]], bancos de logs e sistemas de traces; cada fonte mantém suas próprias capacidades e regras de consulta.

## Exemplo

Um painel conectado ao Prometheus pode exibir a taxa de requisições por segundo. Uma consulta PromQL de exemplo é:

```promql
sum(rate(http_requests_total[5m]))
```

Ela estima a taxa por segundo, somando as séries da métrica nos últimos cinco minutos. O resultado aparece como gráfico se o painel estiver configurado com uma fonte Prometheus.

## Boas práticas

- organize painéis em torno de perguntas operacionais e sinais que ajudem a agir;
- mostre unidades, intervalo e contexto para evitar interpretações erradas;
- crie alertas acionáveis e com limites apropriados, reduzindo notificações repetidas sem utilidade;
- controle quem pode consultar, editar e compartilhar dashboards;
- evite incluir credenciais ou dados sensíveis nas consultas e visualizações.

## Equivalente na AWS

O **Amazon Managed Grafana** é o serviço gerenciado da AWS baseado em Grafana. A AWS opera o ambiente gerenciado e integra fontes de dados AWS e externas; as fontes continuam responsáveis por armazenar os dados consultados.

## Relação com outros assuntos

Grafana pertence à área [[Observability]]. Um fluxo comum combina instrumentação com [[OpenTelemetry]], métricas em [[Prometheus]] e painéis no Grafana.

## Referências

- [Introdução ao Grafana](https://grafana.com/docs/grafana/latest/introduction/)
- [Documentação do Amazon Managed Grafana](https://docs.aws.amazon.com/grafana/latest/userguide/)
