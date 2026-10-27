# O que é o API Server?

O **API Server** (*servidor da API*, normalmente `kube-apiserver`) é a porta de entrada para conversar com o cluster Kubernetes. Ferramentas como `kubectl`, controladores e painéis enviam solicitações a ele para consultar ou alterar objetos.

Uma analogia: é o balcão oficial de um órgão público. Todos entregam pedidos ao balcão, que verifica a identidade, as permissões e se o pedido está correto antes de registrá-lo.

O API Server autentica solicitações, aplica autorização e validações de admissão, expõe a API Kubernetes e persiste o estado no [[etcd]]. Ele não executa diretamente os containers; os nós e seus agentes participam dessa etapa.

## Exemplo

```bash
kubectl get pods -A
```

`kubectl` envia uma consulta ao API Server e mostra os Pods de todos os namespaces para os quais sua identidade tem permissão.

## Boas práticas

- restrinja o acesso de rede ao endpoint da API;
- use identidades individuais ou funções com menor privilégio;
- habilite auditoria e proteja os registros de auditoria;
- não contorne a API alterando diretamente o banco interno do cluster.

## Equivalente na AWS

No **Amazon EKS**, a AWS fornece e opera o endpoint e os servidores do Kubernetes API Server. A equipe controla quem pode alcançá-lo e autenticar-se, mas não administra as máquinas do control plane. Em Kubernetes autogerenciado, a equipe precisa operar e proteger o API Server.

## Relação com outros assuntos

O API Server faz parte do [[Control Plane]] e trabalha junto de [[etcd]] e [[Scheduler]]. A autorização de usuários e grupos é tratada em parte por [[RBAC]].

## Referências

- [Componentes do Kubernetes](https://kubernetes.io/docs/concepts/overview/components/)
- [Acesso ao cluster Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/cluster-auth.html)
