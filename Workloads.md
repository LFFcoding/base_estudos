# Área: Workloads

**Workload** (*carga de trabalho*) é uma aplicação ou tarefa que o cluster precisa executar. No Kubernetes, a aplicação roda dentro de Pods; recursos declarativos dizem quantas cópias devem existir e como elas devem ser mantidas.

Uma analogia: o workload é o serviço de uma cozinha, os Pods são as estações de trabalho e o Kubernetes tenta manter o número de estações necessário para atender os pedidos.

## Conceitos desta área

- [[Pod]]: unidade que reúne um ou mais containers executados juntos;
- [[Deployment]]: mantém Pods de uma aplicação sem estado e gerencia atualizações;
- [[Probes]]: verificações que indicam se o container iniciou, está vivo e pode receber tráfego.

Esses são alguns dos objetos de workload. Existem outros, como StatefulSet, DaemonSet e Job, para tipos diferentes de aplicações e tarefas.

## Como trabalham juntos?

Em geral, descrevemos um Deployment com réplicas de Pods. Cada Pod executa os containers da aplicação. As probes ajudam o Kubernetes a decidir se o Pod deve receber tráfego ou se um container precisa ser reiniciado.

## Boas práticas

- prefira controladores como Deployment em vez de criar Pods avulsos para aplicações contínuas;
- declare solicitações e limites de CPU e memória;
- use probes adequadas ao comportamento real da aplicação;
- mantenha as imagens reproduzíveis e com versões conhecidas.

## Equivalente na AWS

O Amazon EKS executa os mesmos objetos Kubernetes, usando EC2 ou Fargate como capacidade de execução conforme as restrições do workload. Amazon ECS Tasks e Services têm funções parecidas para executar containers, mas seus modelos e APIs não são iguais aos objetos Pod e Deployment.

## Referências

- [Workloads do Kubernetes](https://kubernetes.io/docs/concepts/workloads/)
- [Componentes e workloads no Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/eks-architecture.html)
