# O que é Trivy?

**Trivy** é um scanner de segurança de código aberto que procura vulnerabilidades conhecidas e outros problemas em alvos como [[Container|imagens de containers]], dependências, arquivos de infraestrutura como código e clusters Kubernetes.

Uma analogia: é como verificar os ingredientes e os equipamentos de uma cozinha antes de abrir, procurando itens com alertas conhecidos. Um resultado limpo reduz riscos conhecidos, mas não prova que a aplicação inteira é segura.

## Exemplo

```bash
trivy image nginx:1.27
```

O comando analisa a imagem indicada e apresenta vulnerabilidades encontradas nas dependências que consegue identificar. Em projetos reais, a política deve definir como tratar níveis diferentes de risco.

Também é possível automatizar varreduras de imagens e manifests no [[CI-CD]] antes que sejam publicados ou implantados.

## Boas práticas

- use versões fixadas de ferramentas e imagens;
- execute scans durante o desenvolvimento e antes de produção;
- atualize o banco de vulnerabilidades e reavalie imagens antigas;
- priorize correções por severidade, exposição e disponibilidade de correção, em vez de ignorar todos os alertas ou bloquear sem critério.

## Equivalente na AWS

**Amazon ECR Enhanced Scanning com Amazon Inspector** verifica continuamente imagens armazenadas no ECR e encontra vulnerabilidades cobertas pelas fontes suportadas. Isso se sobrepõe a parte do Trivy, mas não substitui todos os seus scanners de código, configuração, cluster e artefatos.

## Relação com outros assuntos

Trivy pertence a [[Security]] e pode ser usado antes de uma imagem [[Container]] entrar em um cluster Kubernetes. Alertas são tratados como parte das práticas de [[Segurança]].

## Referências

- [Scanner de vulnerabilidades do Trivy](https://trivy.dev/docs/latest/scanner/vulnerability/)
- [Segurança de imagens no Amazon EKS](https://docs.aws.amazon.com/eks/latest/best-practices/image-security.html)
- [Integração do Amazon Inspector com ECR](https://docs.aws.amazon.com/inspector/latest/user/ecr-integration.html)
