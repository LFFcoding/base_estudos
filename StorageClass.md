# O que é StorageClass (Storage Class)?

Uma **StorageClass** descreve uma classe de armazenamento que o cluster oferece e como um volume pode ser provisionado. O nome na imagem aparece como “Storage Class”; o nome do objeto Kubernetes é `StorageClass`.

Uma analogia: é um catálogo com opções de armários — por exemplo, um tipo mais rápido, um compartilhável ou um com determinada política de retenção. A equipe escolhe quais opções oferecer; a aplicação pode pedir uma delas no [[PVC]].

Uma classe costuma declarar um provisionador, parâmetros específicos do driver, política de retenção e quando o volume deve ser vinculado ao Pod. A classe padrão pode ser usada quando o PVC não especifica uma.

## Exemplo conceitual

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: armazenamento-rapido
provisioner: csi.exemplo.invalid
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
```

`csi.exemplo.invalid` é apenas um marcador didático, não um provisionador para instalar. Em um cluster real, o valor e os parâmetros devem corresponder ao driver CSI disponível.

## Boas práticas

- documente capacidade, desempenho, custo, política de retenção e limitações de cada classe;
- tenha cuidado ao escolher uma classe padrão para todos os workloads;
- use `WaitForFirstConsumer` quando o provisionador precisa considerar onde o Pod será agendado;
- teste exclusão, expansão, snapshots e restauração conforme os recursos do driver.

## Equivalente na AWS

No EKS, StorageClass continua sendo um objeto Kubernetes. Seus parâmetros podem solicitar, por exemplo, volumes **Amazon EBS gp3** via CSI ou outro tipo de armazenamento. O provisionador e alguns parâmetros diferem entre o EBS CSI padrão e o EKS Auto Mode; use a documentação correspondente ao modo do cluster.

## Relação com outros assuntos

StorageClass pertence a [[Storage]] e é usada por [[PVC]] para selecionar como um [[PV]] será provisionado.

## Referências

- [StorageClasses no Kubernetes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Criar StorageClass para Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-storage-class.html)
