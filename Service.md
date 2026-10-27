# O que é um Service?

Um **Service** é um objeto Kubernetes que oferece um endereço de rede estável para alcançar um conjunto de Pods. Como os Pods podem ser recriados e receber novos endereços, o Service encaminha o tráfego aos Pods que correspondem ao seu seletor.

Uma analogia: os Pods são funcionários que podem mudar de mesa; o Service é o número do setor, que continua igual e encaminha a chamada para alguém disponível.

Tipos comuns incluem `ClusterIP` (acesso dentro do cluster), `NodePort` (porta em cada nó) e `LoadBalancer` (solicita balanceador de um provedor compatível).

## Exemplo

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

O Service encaminha a porta 80 para a porta 8080 dos Pods com a etiqueta `app: web`.

## Boas práticas

- confira se labels e selector correspondem aos Pods desejados;
- use `ClusterIP` quando o acesso deve ficar apenas dentro do cluster;
- não exponha banco de dados ou serviços internos à internet sem necessidade;
- acompanhe endpoints e readiness dos Pods que recebem tráfego.

## Equivalente na AWS

No EKS, um Service continua sendo recurso Kubernetes. Com AWS Load Balancer Controller, `type: LoadBalancer` normalmente pode provisionar um NLB. `ClusterIP` continua sendo um endereço interno do cluster, não um produto AWS isolado.

## Relação com outros assuntos

Service direciona tráfego a [[Pod|Pods]] normalmente mantidos por um [[Deployment]]. Para rotas HTTP/HTTPS de entrada, pode ser usado com [[Ingress]].

## Referências

- [Services no Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/)
- [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
