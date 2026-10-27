# O que é HPA?

**HPA** (*Horizontal Pod Autoscaler*) é o autoscaler horizontal de Pods do Kubernetes. Ele ajusta o número de réplicas de um recurso escalável, como um [[Deployment]], com base em métricas observadas.

Uma analogia: quando a fila aumenta, o HPA abre mais caixas de atendimento; quando a procura diminui, pode reduzir o número de caixas até o mínimo configurado.

HPA não aumenta CPU ou memória de cada Pod — isso é o objetivo do [[VPA]]. Também não cria automaticamente nós quando falta capacidade no cluster; essa tarefa pode exigir Karpenter ou Cluster Autoscaler.

## Exemplo

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

O HPA precisa receber a métrica configurada. Para métricas de CPU e memória, o cluster normalmente usa Metrics Server, e os containers precisam declarar `requests` adequados.

## Boas práticas

- defina mínimo, máximo e métricas com base em demanda e capacidade;
- inclua o tempo de inicialização da aplicação no planejamento;
- use métricas de negócio ou fila quando CPU não representar bem a carga;
- evite que ferramentas de deploy sobrescrevam o número de réplicas controlado pelo HPA.

## Equivalente na AWS

No EKS, o HPA é o próprio recurso Kubernetes. Em ECS, o **Application Auto Scaling** pode ajustar o número de Tasks de um Service, mas usa outro modelo. No EKS, a camada de nós pode ser expandida separadamente com Karpenter ou Cluster Autoscaler.

## Relação com outros assuntos

HPA pertence a [[Scale & Schedule]] e ajusta réplicas de um [[Deployment]] de [[Pod|Pods]]. [[Prometheus]] pode fornecer métricas personalizadas via adaptadores, quando configurados.

## Referências

- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/)
- [Autoscaling no EKS](https://docs.aws.amazon.com/eks/latest/best-practices/cost-opt-compute.html)
