# Área: Storage

A área **Storage** (*armazenamento*) explica como uma aplicação solicita dados persistentes e como o cluster conecta esse armazenamento aos Pods.

Uma analogia: o Pod é uma sala que pode ser reconstruída; o volume persistente é o arquivo guardado em um armário separado, que continua existindo quando a sala muda.

## Conceitos desta área

- [[PV]]: representa um volume persistente disponível no cluster;
- [[PVC]]: pedido de armazenamento feito por uma aplicação;
- [[StorageClass]]: descreve uma classe e como volumes podem ser provisionados.

## Como se conectam?

Uma pessoa cria um PVC com a capacidade e a classe desejadas. O Kubernetes associa o pedido a um PersistentVolume existente ou solicita a criação dinâmica de um volume por meio do provisionador configurado. O Pod usa o PVC como volume.

## Boas práticas

- escolha armazenamento de bloco ou de arquivos conforme o padrão de acesso;
- entenda a política de retenção antes de excluir claims ou volumes;
- não confunda volume persistente com backup; teste restaurações;
- verifique disponibilidade entre zonas e modo de acesso suportado pelo driver.

## Equivalentes na AWS

No EKS, drivers CSI podem conectar **Amazon EBS** (bloco, normalmente ligado a uma zona) ou **Amazon EFS** (arquivos compartilhados) a PVs, PVCs e StorageClasses. Os serviços AWS fornecem o armazenamento; os objetos Kubernetes ainda descrevem como o workload o solicita e utiliza.

## Referências

- [Volumes persistentes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [StorageClasses](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [Amazon EBS CSI no EKS](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
