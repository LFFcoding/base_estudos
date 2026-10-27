# O que é Ingress?

**Ingress** é um recurso Kubernetes que descreve regras para encaminhar tráfego HTTP/HTTPS externo a [[Service|Services]] do cluster, por exemplo, conforme o domínio ou caminho da URL.

Uma analogia: o Ingress é a recepção de um prédio que lê o destino do visitante e o encaminha ao departamento certo. O recurso descreve as regras, mas precisa de um **Ingress Controller** para implementá-las.

## Exemplo

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  ingressClassName: exemplo-controller
  rules:
    - host: app.exemplo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

`exemplo-controller` é um marcador: substitua pelo nome da classe fornecida pelo controller instalado. Sem um controller compatível, criar o objeto Ingress não configura sozinho um ponto de entrada.

## Estado atual e boas práticas

A API Ingress continua estável, mas está congelada e não recebe novos recursos. A documentação do Kubernetes recomenda considerar **Gateway API** para projetos novos. Se usar Ingress, escolha e mantenha um controller, revise suas anotações específicas e proteja TLS, domínios e exposição pública.

## Equivalente na AWS

No EKS, o AWS Load Balancer Controller pode observar um Ingress e criar um **Application Load Balancer (ALB)** para encaminhar tráfego HTTP/HTTPS. O ALB é o balanceador provisionado; o Ingress continua sendo a declaração Kubernetes.

## Relação com outros assuntos

Ingress pertence a [[Networking]] e encaminha solicitações a um [[Service]]. Para mais opções de roteamento modernas, consulte a [Gateway API do Kubernetes](https://kubernetes.io/docs/concepts/services-networking/gateway/).

## Referências

- [Ingress no Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Roteamento de tráfego com AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
