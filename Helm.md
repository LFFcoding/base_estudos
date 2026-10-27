# O que é Helm?

**Helm** é um gerenciador de pacotes para Kubernetes. Ele organiza recursos relacionados em um **chart**, que pode ser configurado, versionado, instalado e atualizado como uma unidade. Cada instalação gerenciada pelo Helm é chamada de **release**.

Uma analogia: manifests Kubernetes são peças de uma estante; o chart é o projeto que agrupa as peças e as instruções; a release é a estante montada em um ambiente específico.

Helm ajuda a reutilizar templates para recursos como [[Deployment]], [[Service]], [[ConfigMap]] e [[Secret]]. Ele não substitui o Kubernetes: Helm prepara e envia recursos à API, e o cluster continua responsável por reconciliar o estado.

## Comandos de exemplo

```bash
helm lint ./meu-chart
helm template minha-app ./meu-chart -f values-dev.yaml
helm upgrade --install minha-app ./meu-chart -n estudo --create-namespace
```

- `lint` verifica problemas comuns no chart;
- `template` renderiza os manifests para revisão;
- `upgrade --install` instala a release ou atualiza a existente.

Antes de aplicar em produção, revise os manifests renderizados e os valores que serão usados.

## Boas práticas

- fixe e revise versões de charts e dependências;
- separe valores por ambiente sem guardar segredos no repositório;
- mantenha charts pequenos e legíveis;
- teste o resultado renderizado e planeje como fazer rollback de uma release.

## Equivalente na AWS

Não há um serviço AWS que substitua integralmente Helm. É possível usar Helm para instalar aplicações em Amazon EKS; a AWS também oferece alguns add-ons gerenciados, mas eles cobrem apenas componentes específicos. O chart continua sendo a definição da aplicação.

## Relação com outros assuntos

Helm pertence à área [[Config]] e instala recursos no [[Control Plane]]. Ele pode instalar ferramentas como [[Prometheus]] e [[Grafana]], mas não substitui essas ferramentas.

## Referências

- [Introdução oficial ao Helm](https://helm.sh/docs/intro/)
- [Documentação do Helm](https://helm.sh/docs/)
