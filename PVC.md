# O que é um PersistentVolumeClaim (PVC)?

Um **PersistentVolumeClaim (PVC)** é um pedido de armazenamento feito por uma aplicação ou pessoa desenvolvedora. O Kubernetes tenta associar esse pedido a um [[PV]] compatível ou solicitar um volume dinamicamente por uma [[StorageClass]].

Uma analogia: se o PV é um armário disponível, o PVC é o formulário que pede um armário com tamanho e características necessários.

## Exemplo

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dados-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Esse manifest solicita 5 GiB. O modo de acesso suportado depende do driver e do tipo de armazenamento; não escolha um modo sem conferir as características do volume.

Um Pod referencia o claim em `spec.volumes` e monta-o em um caminho do container. Assim, o workload usa o pedido sem depender diretamente dos detalhes do provedor.

## Boas práticas

- defina capacidade e modo de acesso de acordo com o uso real;
- especifique uma classe quando não quiser depender da classe padrão;
- revise impacto da exclusão do PVC e do PV antes de executar comandos;
- faça backup dos dados fora do volume e teste a recuperação.

## Equivalente na AWS

Em EKS, um PVC pode provocar o provisionamento de um volume Amazon EBS pelo EBS CSI Driver ou usar um volume Amazon EFS pelo driver correspondente. O PVC é a solicitação Kubernetes; a cobrança e o ciclo de vida do armazenamento dependem do serviço AWS e da configuração do driver.

## Relação com outros assuntos

PVC pertence a [[Storage]] e liga um [[Pod]] a um [[PV]]. A [[StorageClass]] pode determinar como o volume será criado.

## Referências

- [PersistentVolumeClaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [Usar EBS com EKS](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
