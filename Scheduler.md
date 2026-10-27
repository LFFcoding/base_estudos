# O que é o Scheduler?

O **Scheduler** (*agendador*, `kube-scheduler`) escolhe em qual nó do cluster deve ser executado cada Pod que ainda não tem um nó atribuído.

Uma analogia: é como distribuir caixas entre caminhões. O Scheduler verifica o espaço disponível e as regras de cada caminhão antes de escolher onde cada caixa cabe.

Ele considera informações como recursos pedidos pelo Pod, capacidade dos nós, regras de afinidade e anti-afinidade, restrições e [[Taints]]. O Scheduler escolhe o nó; depois, os componentes desse nó providenciam a execução do Pod. Portanto, agendar não é o mesmo que iniciar o container.

## Exemplo de observação

```bash
kubectl get pods -o wide
kubectl describe pod nome-do-pod
```

O primeiro comando mostra o nó escolhido. O segundo ajuda a investigar eventos e motivos pelos quais um Pod continua `Pending`.

## Boas práticas

- declare `requests` de CPU e memória realistas para ajudar a decisão;
- use afinidade, tolerations e restrições somente quando houver uma necessidade concreta;
- investigue eventos e capacidade do cluster antes de aumentar réplicas;
- não confunda o Scheduler com um autoscaler de nós.

## Equivalente na AWS

No Amazon EKS, o Scheduler padrão do Kubernetes faz parte do control plane gerenciado. Serviços como Karpenter ou Cluster Autoscaler podem adicionar capacidade quando Pods não cabem, mas não substituem o Scheduler: eles ajudam a fornecer nós; o Scheduler ainda posiciona os Pods.

## Relação com outros assuntos

O Scheduler é um componente do [[Control Plane]]. Ele escolhe nós para [[Pod|Pods]] e respeita regras como [[Taints]].

## Referências

- [Kubernetes Scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [Agendamento e preempção](https://kubernetes.io/docs/concepts/scheduling-eviction/)
- [Autoscaling no EKS](https://docs.aws.amazon.com/eks/latest/best-practices/cost-opt-compute.html)
