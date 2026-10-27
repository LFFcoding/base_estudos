# O que é Secret?

Um **Secret** é um objeto Kubernetes destinado a guardar pequenas informações confidenciais, como senhas, tokens e chaves. Um Pod pode receber os valores como arquivo ou variável de ambiente sem incluir o segredo na imagem da aplicação.

Uma analogia: se o [[ConfigMap]] é um quadro com instruções visíveis, Secret é um envelope guardado em um armário com acesso controlado.

Importante: o fato de um valor aparecer codificado em Base64 no YAML não significa que esteja criptografado. Por padrão, o armazenamento interno do Kubernetes pode manter Secrets sem criptografia em repouso; a configuração do cluster e as permissões fazem diferença.

## Exemplo de referência

Este trecho mostra como um container consome uma chave já criada. Ele não contém a senha:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: database-password
```

## Boas práticas

- não envie valores reais de segredo para [[GitHub]] nem os inclua em imagens;
- habilite criptografia em repouso e restrinja leitura por [[RBAC]];
- forneça cada segredo somente aos Pods que precisam dele;
- considere um gerenciador externo e estabeleça rotação e revogação;
- lembre que permissões para criar Pods podem, em alguns cenários, permitir acesso indireto a Secrets do mesmo namespace.

## Equivalente na AWS

O **AWS Secrets Manager** ou parâmetros SecureString do **Systems Manager Parameter Store** podem ser integrados ao EKS por mecanismos como Secrets Store CSI Driver ou External Secrets Operator. São serviços externos ao objeto Secret; verifique sincronização, permissões IAM e comportamento de rotação.

## Relação com outros assuntos

Secret pertence à área [[Config]] e suas proteções fazem parte de [[Segurança]]. Diferente do [[ConfigMap]], seu conteúdo deve ser tratado como confidencial.

## Referências

- [Secrets no Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Boas práticas para Secrets](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)
