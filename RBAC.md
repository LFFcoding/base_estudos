# O que é RBAC?

**RBAC** significa **Role-Based Access Control** (*controle de acesso baseado em papéis*). No Kubernetes, RBAC define quais ações uma identidade pode realizar sobre quais recursos da API.

Uma analogia: o crachá de uma pessoa determina se ela pode apenas consultar uma sala ou também alterar o que há nela. O crachá não é a identidade em si: autenticação confirma quem está fazendo o pedido; RBAC ajuda a decidir se a ação é permitida.

Kubernetes usa `Role` e `RoleBinding` para permissões limitadas a um namespace, e `ClusterRole` e `ClusterRoleBinding` para permissões mais amplas ou reutilizáveis.

## Exemplo

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: leitura-pods
  namespace: estudo
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Essa Role apenas descreve permissões de leitura de Pods no namespace `estudo`; é necessário associá-la a um usuário ou ServiceAccount com um RoleBinding. Não conceda `*` ou `cluster-admin` sem necessidade.

## Boas práticas

- conceda apenas recursos e verbos necessários;
- prefira permissões por namespace quando possível;
- revise bindings e remova identidades antigas;
- verifique permissões efetivas com `kubectl auth can-i` e monitore a auditoria da API.

## Equivalente na AWS

No Amazon EKS, **EKS Access Entries** associam identidades IAM ao acesso ao cluster; a autorização pode usar políticas de acesso do EKS, grupos ligados ao RBAC Kubernetes ou ambos. Permissões IAM para chamar APIs AWS e permissões RBAC para alterar objetos Kubernetes são camadas diferentes.

## Relação com outros assuntos

RBAC pertence a [[Security]] e implementa parte dos controles de [[Segurança]]. Os [[Secret|Secrets]] exigem permissões especialmente restritas.

## Referências

- [Autorização RBAC no Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [EKS Access Entries](https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html)
