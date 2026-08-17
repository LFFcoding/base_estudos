# DevOps

**DevOps** é uma forma de trabalhar que aproxima desenvolvimento (*Development*) e operações (*Operations*). O objetivo é entregar software com mais frequência, segurança e confiabilidade, compartilhando responsabilidades entre as pessoas e as equipes.

DevOps não é um programa específico nem apenas o nome de um cargo. É uma combinação de cultura, práticas, automação e ferramentas.

Uma analogia é uma equipe de restaurante:

- desenvolvimento cria e melhora o cardápio;
- operações mantém a cozinha, os equipamentos e o atendimento funcionando;
- com DevOps, as duas partes trabalham juntas desde o início;
- problemas são percebidos pelo feedback dos clientes e corrigidos rapidamente;
- qualidade e segurança fazem parte de todo o processo, não apenas da inspeção final.

## O problema que DevOps tenta resolver

Quando desenvolvimento e operações trabalham isolados, podem surgir problemas:

- o código funciona no computador de quem desenvolveu, mas não no servidor;
- a equipe de operações descobre mudanças somente no momento da publicação;
- uma publicação manual é esquecida ou executada de forma diferente;
- ninguém sabe claramente quem deve investigar uma falha;
- segurança e monitoramento são deixados para o fim.

DevOps tenta criar um fluxo em que as equipes planejam, constroem, publicam e acompanham o sistema juntas.

## Ciclo DevOps

O trabalho costuma formar um ciclo contínuo:

1. **plan:** planejar uma melhoria;
2. **code:** escrever o código;
3. **build:** compilar e empacotar;
4. **test:** executar testes e verificações;
5. **release:** preparar uma versão;
6. **deploy:** publicar a versão;
7. **operate:** manter o sistema funcionando;
8. **monitor:** observar logs, métricas e comportamento;
9. **feedback:** usar o que foi observado para planejar a próxima melhoria.

Esse ciclo conecta [[Git]], [[CI-CD]], [[Docker]], ambientes e monitoramento.

## Práticas importantes

### Integração contínua

Alterações pequenas são enviadas com frequência para o repositório. O [[CI-CD]] executa compilação, testes e verificações automaticamente.

### Entrega e implantação automatizadas

O processo de publicação é descrito em arquivos e executado de maneira repetível. Isso reduz diferenças entre pessoas e ambientes.

### Infraestrutura como código

A infraestrutura (*Infrastructure as Code* ou IaC) é descrita em arquivos versionados. Em vez de configurar cada servidor manualmente, a equipe registra a configuração e pode recriá-la.

Um arquivo [[Docker Compose]] é um exemplo simples de configuração declarativa: ele descreve serviços, redes, volumes e variáveis.

### Observabilidade

Observabilidade é a capacidade de entender o que está acontecendo dentro de um sistema. Ela usa:

- **logs:** registros de acontecimentos;
- **métricas:** números sobre uso e desempenho;
- **traces:** caminho de uma requisição entre vários serviços;
- **alertas:** avisos quando uma condição importante acontece.

Não basta saber que a aplicação parou. É preciso descobrir por quê, para quem e desde quando.

### Segurança desde o início

Segurança deve fazer parte do ciclo inteiro, prática chamada de *DevSecOps*. Dependências, imagens, permissões, segredos e configurações precisam ser verificados antes da publicação.

### Responsabilidade compartilhada

Quem desenvolve também precisa entender como o sistema executa. Quem opera também participa das decisões que afetam execução, confiabilidade e segurança. O objetivo é evitar a ideia de “isso não é problema meu”.

## Exemplo na stack deste projeto

Um fluxo DevOps possível seria:

1. uma pessoa altera o [[Backend]] Java ou o [[Frontend]] Next.js;
2. cria um commit no [[Git]] e envia uma branch para o [[GitHub]];
3. o [[CI-CD]] executa `./mvnw verify`, lint e build do frontend;
4. o pipeline constrói imagens [[Docker]] com uma versão baseada no commit;
5. os containers são testados em um ambiente separado;
6. uma aprovação libera a publicação;
7. o [[Docker Compose]] atualiza os serviços em uma [[VPS]];
8. o [[Caddy]] encaminha o tráfego para a versão publicada;
9. logs, métricas e health checks verificam a saúde da aplicação;
10. se houver problema, a equipe executa um rollback.

Comandos ilustrativos no servidor:

```bash
docker compose config
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

O primeiro comando valida a configuração. Os seguintes atualizam imagens, iniciam serviços, mostram o estado e exibem logs recentes. Em produção, esses comandos devem fazer parte de um processo controlado, não de uma ação manual sem registro.

## DevOps não significa publicar sem controle

Automatizar não significa remover todas as aprovações. O nível de controle depende do risco:

- uma alteração pequena pode ser publicada automaticamente após os testes;
- uma mudança no banco pode exigir revisão e backup;
- uma alteração de segurança pode exigir aprovação de mais de uma pessoa;
- uma publicação arriscada pode usar canary release ou blue/green deployment.

A automação deve tornar o processo mais seguro, não apenas mais rápido.

## Equivalentes e serviços na AWS

DevOps é uma abordagem, portanto não possui um equivalente único na AWS. A AWS oferece serviços que ajudam a implementar suas práticas:

- **CodePipeline:** coordena etapas de entrega;
- **CodeBuild:** executa compilação e testes;
- **Amazon ECR:** armazena imagens Docker;
- **ECS ou Fargate:** executa containers;
- **CloudWatch:** reúne logs, métricas e alertas;
- **IAM:** controla permissões;
- **CloudFormation:** descreve infraestrutura como código;
- **Systems Manager:** ajuda a administrar instâncias e sessões;
- **AWS Secrets Manager:** armazena segredos.

Também é possível usar GitHub Actions, Terraform ou outras ferramentas para trabalhar com a AWS.

As diferenças principais são:

- em uma VPS, a equipe administra mais diretamente o sistema operacional e pode usar Docker Compose;
- na AWS, serviços gerenciados reduzem parte da administração, mas exigem configurar vários recursos e permissões;
- a AWS oferece integração e escala maiores, mas a cobrança e a arquitetura podem ficar mais complexas;
- as ferramentas não substituem os princípios DevOps: colaboração, automação, feedback, segurança e responsabilidade compartilhada continuam necessários.

## Boas práticas

- fazer mudanças pequenas e frequentes;
- manter código, infraestrutura e pipeline versionados;
- automatizar build, testes e verificações de segurança;
- construir uma vez e promover o mesmo artefato entre ambientes;
- não guardar segredos no código, nas imagens ou nos logs;
- usar permissões mínimas para pessoas, pipelines e serviços;
- definir health checks, métricas, logs e alertas antes da produção;
- ter rollback documentado e testado;
- testar backups e restauração do [[PostgreSQL]];
- revisar custos e capacidade dos ambientes;
- compartilhar o conhecimento de operação com a equipe de desenvolvimento;
- fazer post-mortems sem culpabilização depois de incidentes, procurando melhorar o sistema;
- medir resultados, como frequência de publicação, tempo de recuperação e quantidade de falhas.

## Resumo

> **DevOps é uma forma colaborativa e automatizada de construir, publicar e operar sistemas com segurança e confiabilidade.**

Ele conecta pessoas, processos e ferramentas para que o software continue melhorando depois que o código é escrito.
