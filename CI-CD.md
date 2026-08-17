# CI/CD

**CI/CD** é um conjunto de práticas de [[DevOps]] para automatizar a construção, a verificação e a publicação de aplicações.

- **CI (*Continuous Integration* ou integração contínua):** juntar alterações ao projeto com frequência e verificar automaticamente se elas compilam e passam nos testes;
- **CD (*Continuous Delivery* ou entrega contínua):** deixar uma versão validada pronta para ser publicada quando houver aprovação;
- **CD (*Continuous Deployment* ou implantação contínua):** publicar automaticamente uma versão que passou pelas verificações.

Uma analogia é uma linha de produção:

- a pessoa desenvolvedora entrega uma mudança;
- a esteira compila e testa o produto;
- inspetores verificam qualidade e segurança;
- o produto aprovado é embalado;
- a equipe decide para qual loja ele será enviado ou o envio acontece automaticamente.

CI/CD não é apenas “publicar automaticamente”. A ideia é criar um caminho repetível, com verificações e registros, desde o código até a aplicação em execução.

## Por que usar CI/CD?

CI/CD ajuda a:

- encontrar erros mais cedo;
- reduzir tarefas manuais e repetitivas;
- manter o processo de publicação igual para todas as pessoas;
- saber qual versão está em cada ambiente;
- diminuir o risco de uma publicação;
- voltar para uma versão anterior quando necessário;
- entregar melhorias menores com mais frequência.

## Pipeline

Um pipeline (*pipeline*) é a sequência automatizada de etapas. Um fluxo comum é:

1. **código:** uma alteração é enviada ao [[GitHub]];
2. **build:** o sistema compila o [[Backend]] e constrói o [[Frontend]];
3. **testes:** testes unitários, de integração e de interface são executados;
4. **qualidade e segurança:** o código e as dependências são analisados;
5. **artefato:** é gerado um pacote ou uma imagem [[Docker]] com uma versão identificável;
6. **ambiente de teste:** o artefato é executado para validações adicionais;
7. **aprovação:** uma pessoa ou regra autoriza a publicação, quando necessário;
8. **produção:** a aplicação é publicada em uma [[VPS]] ou em serviços de nuvem;
9. **verificação:** health checks, logs e métricas confirmam se a versão está funcionando.

Se uma etapa importante falhar, o pipeline deve parar. Isso impede que uma versão com problema avance silenciosamente.

## Termos importantes

- **Runner:** computador ou ambiente que executa as etapas do pipeline;
- **Artifact:** arquivo produzido pelo build, como um `.jar` ou uma imagem Docker;
- **Environment:** ambiente como desenvolvimento, teste, homologação ou produção;
- **Secret:** informação sensível, como token, senha ou chave privada;
- **Release:** versão preparada para ser disponibilizada;
- **Rollback:** retorno para uma versão anterior conhecida por funcionar;
- **Smoke test:** teste rápido que verifica se as funções básicas estão vivas;
- **Approval gate:** ponto que exige aprovação antes de continuar.

## Exemplo com GitHub Actions

O GitHub Actions é uma ferramenta de automação que pode implementar um pipeline. Um arquivo `.github/workflows/ci.yml` poderia ser:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - name: Baixar o código
        uses: actions/checkout@v4

      - name: Preparar Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"
          cache: maven

      - name: Executar verificações
        run: ./mvnw verify
```

Esse exemplo:

- executa quando um pull request ou um push para `main` acontece;
- cria um ambiente de execução para o job;
- baixa o código;
- prepara Java 17;
- usa o cache do Maven;
- executa o ciclo `verify`, que deve compilar e testar o backend.

Se o projeto não possuir Maven Wrapper, use uma instalação controlada do Maven, como `mvn verify`, e documente a versão esperada.

Um job semelhante pode preparar o frontend:

```yaml
  frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run build
```

O comando `npm ci` instala exatamente as versões registradas no arquivo de lock, tornando o build mais previsível.

## Publicação com Docker

Depois dos testes, o pipeline pode construir uma imagem com uma versão baseada no commit:

```bash
docker build -t backend:${GIT_COMMIT_SHA} .
docker push registry.exemplo.com/backend:${GIT_COMMIT_SHA}
```

A variável `GIT_COMMIT_SHA` representa o identificador do commit. Usar uma versão imutável ajuda a saber exatamente qual código foi publicado.

No servidor, um fluxo simples pode ser:

```bash
docker compose config
docker compose pull
docker compose up -d
```

Esses comandos devem ser executados em um ambiente controlado, com permissões adequadas e uma estratégia de rollback. Não use tags genéricas como `latest` para descobrir qual versão está em produção.

## CI/CD e ambientes

É comum separar ambientes:

- **desenvolvimento:** usado para criar e experimentar;
- **teste:** usado pelo pipeline para validar a aplicação;
- **homologação (*staging*):** parecido com produção, usado para uma última verificação;
- **produção:** ambiente utilizado pelos usuários.

O mesmo artefato deve ser promovido entre ambientes sempre que possível. Construir novamente o código para cada ambiente pode gerar diferenças difíceis de investigar.

## Segurança no pipeline

O pipeline pode acessar código, servidores e dados importantes. Por isso:

- nunca coloque segredos diretamente no arquivo YAML;
- use secrets do provedor ou um gerenciador de segredos;
- dê ao pipeline somente as permissões necessárias;
- prefira credenciais temporárias e [[OIDC|OIDC (OpenID Connect)]] quando o provedor permitir;
- não imprima tokens, senhas ou chaves nos logs;
- revise alterações em arquivos de pipeline como revisaria código;
- fixe versões de actions, imagens e dependências;
- escaneie imagens e bibliotecas contra vulnerabilidades;
- exija aprovação para produção quando o risco justificar;
- proteja a branch principal e exija testes antes do merge.

## Rollback

Uma publicação pode falhar mesmo depois de passar pelos testes. Por isso, cada versão precisa ser identificável e o rollback deve ser planejado.

Um rollback pode significar:

- apontar o serviço para a imagem anterior;
- restaurar o pacote `.jar` anterior;
- reverter uma configuração;
- retornar o tráfego para uma versão antiga durante uma implantação gradual.

Rollback de código não desfaz automaticamente alterações destrutivas no banco de dados. Migrações devem ser planejadas para permitir recuperação ou compatibilidade entre versões.

## CI/CD na AWS

Na AWS, uma solução equivalente pode combinar:

- **AWS CodePipeline:** coordena as etapas do pipeline;
- **AWS CodeBuild:** compila e executa testes;
- **Amazon ECR:** armazena imagens Docker;
- **Amazon ECS ou AWS Fargate:** executa os containers;
- **Amazon CloudWatch:** reúne logs e métricas;
- **AWS CodeDeploy:** pode ajudar em estratégias como blue/green, dependendo do serviço usado.

Também é possível usar GitHub Actions com OIDC para publicar na AWS sem guardar uma chave permanente no GitHub.

As diferenças principais são:

- GitHub Actions fica integrado ao repositório do GitHub e é flexível para várias plataformas;
- CodePipeline, CodeBuild e os demais serviços são integrados ao ecossistema AWS;
- a AWS pode reduzir a quantidade de integrações externas, mas exige configurar IAM, rede, logs e cobrança dos serviços;
- GitHub Actions pode publicar em uma [[VPS]], enquanto os serviços AWS podem executar a aplicação em ECS, Fargate ou outros ambientes;
- nenhuma ferramenta elimina a necessidade de testes, segurança, monitoramento e rollback.

## Boas práticas

- executar o pipeline em todo pull request;
- bloquear o merge quando testes essenciais falharem;
- manter pipelines rápidos, separando verificações rápidas das mais demoradas;
- armazenar artefatos com versões imutáveis;
- construir uma vez e promover o mesmo artefato entre ambientes;
- incluir testes unitários, integração e smoke tests adequados ao risco;
- revisar dependências e imagens regularmente;
- usar ambientes separados e permissões mínimas;
- ter aprovação e rollback para produção;
- monitorar a aplicação depois da publicação;
- documentar como investigar falhas e recuperar uma versão anterior;
- evitar que o pipeline dependa de arquivos ou configurações manuais fora do repositório.

## Resumo

> **CI/CD é uma prática de automatizar a verificação, a preparação e a publicação de aplicações com segurança e repetibilidade.**

CI verifica as mudanças continuamente; CD deixa a entrega pronta ou publica automaticamente, dependendo da política adotada.
