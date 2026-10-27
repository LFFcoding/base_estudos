# O que é Cilium?

**Cilium** é um projeto de rede, segurança e observabilidade para ambientes com containers, muito usado como plugin CNI em Kubernetes. Ele usa **eBPF** do kernel Linux para implementar funções como conectividade, balanceamento e aplicação de políticas de rede.

Uma analogia: se a rede do cluster é um conjunto de ruas, Cilium instala regras e sensores próximos às entradas para encaminhar e observar o tráfego.

Cilium não é um objeto nativo do Kubernetes. É um componente que precisa ser instalado, configurado e atualizado. Com Hubble, seu ecossistema também pode mostrar relações e fluxos de comunicação entre workloads.

## Exemplo de verificação

```bash
cilium status
```

Esse comando, disponível quando a CLI do Cilium está instalada, mostra o estado dos componentes. Em um cluster que não usa Cilium, o comando não está disponível.

## Boas práticas

- valide compatibilidade de kernel, plataforma e versão antes de instalar;
- teste políticas de rede em ambiente não produtivo antes de aplicá-las amplamente;
- limite privilégios do agente e monitore seu estado;
- entenda qual CNI está ativo: alterar o plugin de rede de um cluster exige planejamento.

## Equivalente na AWS

O **Amazon VPC CNI** é o plugin de rede comum para Pods em Amazon EKS e é a aproximação AWS para a função CNI; ele não oferece exatamente o mesmo conjunto de políticas e visibilidade do Cilium. A documentação do EKS admite plugins alternativos em cenários compatíveis, mas a escolha precisa considerar suporte, operação e arquitetura do cluster.

## Relação com outros assuntos

Cilium pertence a [[Networking]], pode aplicar políticas relacionadas à [[Security]] e fornece visibilidade que complementa [[Observability]].

## Referências

- [Visão geral de Cilium e Hubble](https://docs.cilium.io/en/stable/overview/intro/)
- [eBPF no Cilium](https://docs.cilium.io/en/stable/network/ebpf/intro/)
- [Amazon VPC CNI](https://docs.aws.amazon.com/eks/latest/best-practices/vpc-cni.html)
