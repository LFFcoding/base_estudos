# Área: Config

A área **Config** reúne formas de separar as configurações da aplicação de seu código e de organizar a instalação de recursos do Kubernetes.

Uma analogia: a imagem do container é uma receita; a configuração informa ingredientes locais, como endereço de serviço e opções do ambiente, sem precisar criar uma receita diferente para cada cozinha.

## Conceitos desta área

- [[ConfigMap]]: armazena configurações que não são sigilosas;
- [[Secret]]: guarda dados sensíveis, como tokens e senhas, com controles adicionais;
- [[Helm]]: empacota e instala conjuntos de recursos Kubernetes por meio de charts.

## Separar responsabilidades

Uma mesma imagem da aplicação pode ser usada em desenvolvimento, teste e produção. O ambiente fornece valores diferentes por ConfigMap, Secret ou mecanismos externos. Helm pode agrupar os manifests necessários para criar e atualizar os recursos.

## Boas práticas

- não coloque senhas, tokens ou chaves em ConfigMaps;
- restrinja a leitura de Secrets e configure criptografia em repouso;
- versione manifests e charts, mas nunca segredos reais;
- revise os valores renderizados do Helm antes de aplicar uma mudança.

## Equivalentes na AWS

O **AWS Systems Manager Parameter Store** e o **AWS AppConfig** ajudam a armazenar ou distribuir configurações; o **AWS Secrets Manager** é próprio para segredos. Esses serviços não são cópias idênticas dos objetos Kubernetes. Helm pode ser usado com Amazon EKS, mas a AWS não oferece um serviço gerenciado que substitua todas as funções de Helm.

## Referências

- [Configuração no Kubernetes](https://kubernetes.io/docs/concepts/configuration/)
- [Introdução ao Helm](https://helm.sh/docs/intro/)
