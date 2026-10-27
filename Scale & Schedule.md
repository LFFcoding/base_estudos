# Área: Scale & Schedule

**Scale & Schedule** (*escala e agendamento*) reúne decisões sobre quantas cópias de uma aplicação devem rodar e em quais nós elas podem ser colocadas.

Uma analogia: escalar é abrir ou fechar caixas de atendimento conforme a fila; agendar é escolher em qual sala cada atendente vai trabalhar, respeitando equipamentos e regras do local.

## Conceitos desta área

- [[HPA]]: ajusta horizontalmente o número de Pods;
- [[VPA]]: recomenda ou ajusta recursos de CPU e memória por Pod;
- [[Taints]]: marcam nós para afastar Pods que não declaram tolerância compatível.

Escalar Pods não cria automaticamente mais máquinas. Um cluster também pode precisar de um autoscaler de nós, como Karpenter ou Cluster Autoscaler, para fornecer capacidade.

## Boas práticas

- escolha métricas ligadas à demanda real, não somente à CPU por conveniência;
- defina requests de recursos para que o agendamento e autoscaling funcionem bem;
- configure limites mínimos, máximos e comportamentos de redução;
- teste taints e tolerations para evitar Pods sem lugar ou em nós inadequados.

## Equivalente na AWS

No EKS, HPA e VPA continuam sendo recursos Kubernetes; a AWS recomenda combiná-los com ajuste de capacidade dos nós, por exemplo usando Karpenter ou Cluster Autoscaler. Para ECS, o Application Auto Scaling pode ajustar o número de Tasks, mas não tem correspondência direta com VPA ou taints do Kubernetes.

## Referências

- [Autoscaling de workloads Kubernetes](https://kubernetes.io/docs/concepts/workloads/autoscaling/)
- [Taints e tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Computação e autoscaling no EKS](https://docs.aws.amazon.com/eks/latest/best-practices/cost-opt-compute.html)
