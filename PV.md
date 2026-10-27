# O que é um PersistentVolume (PV)?

Um **PersistentVolume (PV)** é um recurso Kubernetes que representa armazenamento persistente disponível para workloads. O volume tem ciclo de vida separado do [[Pod]], então os dados não precisam desaparecer quando um Pod é substituído.

Uma analogia: o PV é um armário de arquivo identificado e disponível para uso; a aplicação solicita acesso a ele por meio de um [[PVC]].

Um PV pode ser criado por uma pessoa administradora ou provisionado dinamicamente por um driver de armazenamento. O recurso registra características como capacidade, modo de acesso, classe e política de retenção.

## Fluxo básico

```text
StorageClass (opcional) → PVC solicita espaço → PV é associado → Pod monta o PVC
```

O PV não é, por si só, uma cópia de segurança. Se os dados forem importantes, defina uma estratégia de backup e teste a restauração.

## Boas práticas

- escolha e revise a política de recuperação (*reclaim policy*) antes de excluir recursos;
- entenda limitações de região/zona, acesso e expansão do driver;
- não confunda persistência com backup;
- monitore capacidade e faça testes de restauração.

## Equivalente na AWS

No EKS, o driver CSI do **Amazon EBS** pode fornecer volumes de bloco como PVs; o **Amazon EFS CSI** pode fornecer armazenamento de arquivos compartilhável. EBS e EFS são os serviços de armazenamento subjacentes, enquanto PV continua sendo a abstração Kubernetes.

## Relação com outros assuntos

PV faz parte de [[Storage]], pode ser solicitado por [[PVC]] e pode ser criado dinamicamente a partir de uma [[StorageClass]].

## Referências

- [Volumes persistentes no Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Amazon EBS CSI no EKS](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [Amazon EFS CSI no EKS](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)
