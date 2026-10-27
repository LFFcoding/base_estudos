# O que são Taints?

**Taints** (*manchas ou marcações de repulsão*) são marcações aplicadas a nós para impedir que Pods sejam agendados neles, a menos que os Pods tenham uma toleration compatível.

Uma analogia: um nó coloca uma placa “acesso restrito”; só entram workloads que trazem a autorização correspondente. A toleration permite a entrada, mas não obriga o Scheduler a escolher aquele nó.

## Exemplo

```bash
kubectl taint nodes worker-1 tipo=especial:NoSchedule
```

Esse comando impede novos Pods sem toleration correspondente de serem agendados em `worker-1`. Para permitir um Pod específico, o template dele pode incluir:

```yaml
tolerations:
  - key: tipo
    operator: Equal
    value: especial
    effect: NoSchedule
```

Para que esse Pod também seja direcionado ao nó especial, use uma regra complementar, como node affinity ou `nodeSelector`.

## Efeitos comuns

- `NoSchedule`: impede novos agendamentos que não toleram a marcação;
- `PreferNoSchedule`: tenta evitar o nó, mas não é uma proibição absoluta;
- `NoExecute`: também pode remover Pods existentes que não toleram o taint.

## Boas práticas

- documente o motivo de cada taint e a equipe responsável;
- aplique tolerations apenas aos workloads que realmente precisam;
- combine taints com afinidade se quiser garantir direcionamento;
- use cautela com `NoExecute`, que pode expulsar Pods em execução.

## Equivalente na AWS

Taints são uma função do Scheduler Kubernetes e continuam existindo em nós do EKS. Grupos de nós gerenciados e Karpenter podem criar ou configurar capacidade, mas não substituem o comportamento de taints e tolerations; essas políticas devem ser declaradas no cluster.

## Relação com outros assuntos

Taints pertencem a [[Scale & Schedule]] e são avaliados pelo [[Scheduler]]. Tolerations são configuradas no Pod; não são o mesmo que uma permissão de [[RBAC]].

## Referências

- [Taints e tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Autoscaling e capacidade do EKS](https://docs.aws.amazon.com/eks/latest/best-practices/cost-opt-compute.html)
