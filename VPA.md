# O que é VPA?

**VPA** (*Vertical Pod Autoscaler*) é um autoscaler que recomenda ou ajusta os recursos de CPU e memória solicitados pelos Pods de um workload. Diferentemente do [[HPA]], que altera a quantidade de Pods, o VPA tenta dimensionar cada réplica.

Uma analogia: o HPA abre mais mesas; o VPA troca a mesa por uma maior para dar mais espaço a cada atendente.

O VPA não faz parte do conjunto padrão instalado em todo cluster Kubernetes: normalmente precisa ser instalado separadamente. Dependendo do modo de atualização e das capacidades do cluster, aplicar novos recursos pode exigir recriar Pods ou afetar a disponibilidade.

## Exemplo conceitual

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: api
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  updatePolicy:
    updateMode: "Off"
```

Com `Off`, o objeto pode calcular recomendações sem aplicá-las automaticamente. O VPA precisa estar instalado para que esse recurso seja reconhecido.

## Boas práticas

- comece observando recomendações antes de ativar atualizações automáticas;
- verifique limites de CPU e memória e evite competir com HPA sobre a mesma métrica;
- avalie interrupção de Pods e impacto na disponibilidade;
- acompanhe a qualidade das recomendações conforme carga e sazonalidade mudam.

## Equivalente na AWS

No EKS, pode-se instalar e operar o VPA como componente Kubernetes, sujeito à configuração do cluster. A AWS recomenda usá-lo junto de HPA e autoscaling de nós quando adequado; não há um serviço AWS que seja um substituto direto e automático do VPA. ECS Application Auto Scaling ajusta Tasks, mas não dimensiona recursos por Task da mesma forma.

## Relação com outros assuntos

VPA pertence a [[Scale & Schedule]] e ajusta recursos de [[Pod|Pods]] gerenciados por um [[Deployment]]. Compare com [[HPA]], que altera o número de réplicas.

## Referências

- [Vertical Pod Autoscaler](https://kubernetes.io/docs/concepts/workloads/autoscaling/vertical-pod-autoscale/)
- [Computação e autoscaling no EKS](https://docs.aws.amazon.com/eks/latest/best-practices/cost-opt-compute.html)
