# O que é um Pod?

Um **Pod** é a menor unidade que o Kubernetes agenda para execução. Ele contém um ou mais containers que compartilham o mesmo endereço de rede e podem compartilhar volumes.

Uma analogia: um Pod é como uma pequena sala de trabalho. Os containers dentro dela trabalham próximos e compartilham alguns recursos; Pods diferentes são salas separadas.

Na maioria das aplicações, um Pod contém um container principal. Às vezes, containers auxiliares ficam no mesmo Pod quando precisam acompanhar de perto o ciclo de vida da aplicação. Um Pod é substituível: se falhar, um controlador como [[Deployment]] pode criar outro, que terá uma nova identidade e possivelmente outro endereço IP.

## Exemplo mínimo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: exemplo
spec:
  containers:
    - name: web
      image: nginx:1.27
```

Esse manifest pede um Pod com um container baseado na imagem indicada. Em aplicações contínuas, normalmente declare Pods por meio de um controlador em vez de criar Pods avulsos.

## Boas práticas

- prefira [[Deployment]] para aplicações sem estado;
- não guarde dados importantes apenas no disco temporário do Pod;
- defina recursos e [[Probes]] adequados;
- evite juntar containers sem necessidade de compartilhar rede, volumes e ciclo de vida.

## Equivalente na AWS

No EKS, o Pod continua sendo um Pod Kubernetes, executado em nós EC2 ou, sob condições compatíveis, em Fargate. Uma ECS Task é uma unidade de execução de containers com finalidade parecida, mas não tem o mesmo contrato nem os mesmos objetos de rede e controle.

## Relação com outros assuntos

Pods pertencem à área [[Workloads]], recebem configuração de [[ConfigMap]] e [[Secret]], podem ser expostos por [[Service]] e solicitar armazenamento com [[PVC]].

## Referências

- [Pods no Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Componentes do Kubernetes](https://kubernetes.io/docs/concepts/overview/components/)
