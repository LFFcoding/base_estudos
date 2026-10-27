# O que são Probes?

**Probes** (*sondas de saúde*) são verificações periódicas que o kubelet executa para decidir se um container já iniciou, está funcionando ou pode receber tráfego.

Uma analogia: são perguntas diferentes feitas a uma loja: “já abriu?”, “a loja está funcionando?” e “pode receber clientes agora?”. Uma resposta errada pode fazer o Kubernetes reiniciar um container ou retirar temporariamente o Pod do tráfego.

Existem três tipos principais:

- **startup probe:** dá tempo para a aplicação terminar de iniciar;
- **liveness probe:** indica se o container está travado e deve ser reiniciado;
- **readiness probe:** indica se o Pod está pronto para receber requisições.

## Exemplo

```yaml
readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  periodSeconds: 5
```

Esse trecho verifica periodicamente o endpoint de prontidão da aplicação. O endpoint e a porta são exemplos: precisam existir e refletir a saúde real do serviço.

## Boas práticas

- não use liveness para verificar indisponibilidade temporária de um serviço externo; reinícios em cascata podem piorar a falha;
- escolha tempos, limites e endpoints compatíveis com a inicialização e a carga;
- se a aplicação demora para iniciar, configure startup probe;
- readiness deve deixar de passar quando o Pod não puder atender tráfego.

## Equivalente na AWS

No EKS, as probes Kubernetes funcionam normalmente. ECS container health checks e health checks de ALB/NLB têm objetivos relacionados, mas atuam em camadas diferentes. Health checks do balanceador podem complementar, não substituir, as probes da aplicação.

## Relação com outros assuntos

Probes configuram-se no container de um [[Pod]] ou de um [[Deployment]] e influenciam quais Pods recebem tráfego de um [[Service]].

## Referências

- [Liveness, readiness e startup probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/)
- [Configurar probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)
