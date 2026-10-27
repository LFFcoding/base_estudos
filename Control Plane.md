# Área: Control Plane

O **control plane** (*plano de controle*) é a parte do Kubernetes que recebe pedidos sobre o cluster, guarda o estado desejado e coordena as decisões necessárias para aproximar o estado real desse objetivo.

Uma analogia: é como a equipe de controle de um aeroporto. Ela recebe os planos, mantém o registro dos voos e decide para onde cada aeronave deve ir; não pilota cada avião diretamente.

## Conceitos desta área

- [[API Server]]: entrada principal para consultar e alterar objetos do cluster;
- [[etcd]]: armazenamento do estado persistente do cluster;
- [[Scheduler]]: escolhe em qual nó um Pod ainda sem destino deve executar.

O control plane também contém outros componentes, como o controller manager, que não estão entre os 24 conceitos destacados na imagem. Em conjunto, esses componentes observam o estado declarado e trabalham para reconciliá-lo.

## Exemplo do fluxo

Quando alguém cria uma aplicação, o pedido chega ao API Server e é registrado no etcd. O Scheduler observa os Pods que ainda precisam de um nó e escolhe um local adequado. Outros componentes continuam acompanhando o estado para corrigir diferenças.

## Boas práticas

- use as APIs do Kubernetes em vez de alterar diretamente os dados internos do cluster;
- restrinja quem pode acessar o API Server e proteja suas credenciais;
- planeje cópias de segurança e recuperação do estado do cluster;
- em produção, monitore disponibilidade e latência do control plane.

## Equivalente na AWS

O **Amazon EKS** é a opção gerenciada para executar Kubernetes. A AWS opera o control plane — incluindo API Server e etcd — enquanto a equipe configura o cluster e decide como executar as aplicações. Em um cluster autogerenciado em [[VPS]] ou EC2, a equipe assume também a instalação, disponibilidade, atualização e recuperação desses componentes. O serviço gerenciado reduz esse trabalho, mas não elimina a responsabilidade de configurar acesso, rede e workloads corretamente.

## Referências

- [Componentes do Kubernetes](https://kubernetes.io/docs/concepts/overview/components/)
- [Arquitetura do Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html)
