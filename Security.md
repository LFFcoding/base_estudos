# Área: Security

A área **Security** (*segurança*) cobre prevenção, controle de acesso e detecção de comportamentos suspeitos em clusters e aplicações.

Uma analogia: a segurança de um prédio combina inspeção do material que entra, crachás que limitam onde cada pessoa pode ir e alarmes que detectam atividade estranha.

## Conceitos desta área

- [[Trivy]]: procura vulnerabilidades e problemas de configuração em imagens e artefatos;
- [[Falco]]: observa eventos durante a execução e pode alertar sobre comportamentos suspeitos;
- [[RBAC]]: define quais identidades podem realizar quais ações na API Kubernetes.

Essas ferramentas cumprem funções diferentes: escanear antes da implantação, detectar eventos em runtime e autorizar operações. Uma não substitui as demais.

## Boas práticas

- aplique o menor privilégio a usuários, workloads e ferramentas;
- verifique imagens e manifests antes de implantá-los;
- proteja Secrets e audite acessos ao API Server;
- trate alertas como sinais para investigação e mantenha um processo de atualização e resposta.

## Equivalentes na AWS

Amazon ECR com **Amazon Inspector** pode verificar vulnerabilidades de imagens; **GuardDuty Runtime Monitoring** oferece detecção de ameaças em workloads compatíveis; e **IAM com EKS Access Entries**, junto de RBAC Kubernetes quando aplicável, integra identidades AWS ao acesso ao cluster. Há sobreposição de finalidade, mas nenhum desses serviços é uma cópia exata de Trivy, Falco e RBAC.

## Referências

- [Boas práticas de segurança de imagens no EKS](https://docs.aws.amazon.com/eks/latest/best-practices/image-security.html)
- [RBAC no Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [GuardDuty Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html)
