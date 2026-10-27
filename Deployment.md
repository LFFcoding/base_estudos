# O que é um Deployment?

Um **Deployment** é um recurso Kubernetes que declara como manter e atualizar um conjunto de Pods de uma aplicação. Ele cria e controla ReplicaSets, que mantêm o número desejado de Pods.

Uma analogia: é o gerente de uma equipe. Se o objetivo é manter três atendentes trabalhando, ele percebe quando há menos e organiza a reposição. Também coordena a troca gradual de uma versão da aplicação por outra.

## Exemplo

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
```

O Deployment mantém três réplicas que correspondem ao seletor `app: web`. Para atualização em produção, a estratégia e as condições de prontidão precisam ser configuradas e observadas.

## Boas práticas

- mantenha o manifest versionado e declare apenas o estado desejado;
- use etiquetas e seletores que correspondam exatamente;
- configure recursos e [[Probes]] para orientar agendamento e tráfego;
- acompanhe rollout e tenha um plano de recuperação; mudar a imagem não garante que a nova versão esteja saudável.

## Equivalente na AWS

No Amazon EKS, o Deployment é o mesmo recurso Kubernetes. Um ECS Service com desired count e atualização de Tasks tem finalidade semelhante, mas o modelo operacional não é idêntico. Em qualquer plataforma, a implantação gradual precisa observar a saúde da aplicação.

## Relação com outros assuntos

Deployment pertence a [[Workloads]], cria [[Pod|Pods]] e pode ser exposto por um [[Service]]. O [[HPA]] pode ajustar seu número de réplicas.

## Referências

- [Deployments no Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Boas práticas de configuração de workloads](https://kubernetes.io/docs/concepts/configuration/overview/)
