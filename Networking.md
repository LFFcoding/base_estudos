# Área: Networking

**Networking** (*redes*) trata de como Pods, aplicações e usuários conseguem trocar dados dentro e fora do cluster.

Uma analogia: os Pods são salas que podem ser substituídas ou mudar de endereço. A rede fornece endereços estáveis, caminhos e regras para que a comunicação continue funcionando.

## Conceitos desta área

- [[Service]]: oferece um endereço estável para um conjunto de Pods;
- [[Ingress]]: descreve rotas HTTP/HTTPS de entrada para Services;
- [[Cilium]]: pode fornecer rede, políticas e visibilidade usando eBPF.

Kubernetes define APIs de rede, mas parte do comportamento depende de plugins e controladores instalados no cluster. Ingress continua disponível, porém sua API está congelada; para recursos novos, o projeto Kubernetes recomenda avaliar Gateway API.

## Boas práticas

- exponha externamente apenas os serviços necessários;
- use políticas de rede para limitar comunicações quando o plugin instalado as suporta;
- proteja tráfego com TLS quando atravessar redes não confiáveis;
- planeje endereços IP, DNS, balanceamento e observabilidade desde o início.

## Equivalentes na AWS

No EKS, o **Amazon VPC CNI** fornece a rede dos Pods; o **AWS Load Balancer Controller** pode criar ALBs para Ingress e NLBs para Services do tipo `LoadBalancer`. São integrações com a rede Kubernetes, não substitutos exatos de todos os seus objetos. O uso do Cilium depende da arquitetura e da compatibilidade escolhidas para o cluster.

## Referências

- [Services, balanceamento e rede no Kubernetes](https://kubernetes.io/docs/concepts/services-networking/)
- [Ingress e Gateway API](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Rede do Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html)
