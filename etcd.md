# O que é etcd?

**etcd** é um armazenamento distribuído de chave e valor usado pelo Kubernetes para guardar o estado persistente do cluster. O [[API Server]] consulta e atualiza esses dados quando objetos Kubernetes são criados, modificados ou removidos.

Uma analogia: é o livro de registro oficial do cluster. Ele anota que aplicações foram declaradas, quais configurações existem e outros dados necessários para que o control plane saiba qual estado deve manter.

O etcd guarda informações de controle do cluster; não é o banco de dados normal da aplicação. Dados de negócio devem ficar em serviços apropriados, como [[PostgreSQL]].

## Boas práticas

- proteja o tráfego e o armazenamento com controles de acesso e criptografia;
- em clusters autogerenciados, faça snapshots consistentes e teste a restauração;
- limite acesso direto ao etcd; use a API Kubernetes para administrar recursos;
- monitore espaço, latência e saúde dos membros do cluster.

## Equivalente na AWS

No Amazon EKS, o etcd faz parte do control plane gerenciado pela AWS. Não há um serviço que exponha esse etcd como um banco que a equipe do cluster deva administrar; a equipe gerencia os objetos Kubernetes pela API. Em um cluster autogerenciado, a operação, backup e recuperação do etcd ficam sob responsabilidade da equipe.

## Relação com outros assuntos

O etcd integra o [[Control Plane]] e recebe mudanças por meio do [[API Server]]. O [[Scheduler]] e outros controladores observam o estado exposto pela API, não devem editar diretamente o armazenamento.

## Referências

- [Arquitetura do cluster Kubernetes](https://kubernetes.io/docs/concepts/architecture/)
- [Documentação oficial do etcd](https://etcd.io/docs/)
- [Arquitetura do Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html)
