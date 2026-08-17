# Docker

**Docker** é uma plataforma para empacotar e executar aplicações em **containers**. Um container reúne a aplicação e parte dos arquivos e dependências necessários para que ela funcione de maneira previsível.

Uma analogia é o transporte de mercadorias:

- a aplicação é a mercadoria;
- o container é uma caixa padronizada;
- a imagem é o modelo pronto dessa caixa;
- o Docker é o sistema que cria, transporta e executa as caixas;
- o computador ou servidor é o local onde as caixas são abertas e executadas.

Com Docker, o [[Backend]], o [[Frontend]] e o [[PostgreSQL]] podem ser executados em ambientes separados, cada um com suas próprias dependências.

## Por que usar Docker?

Docker ajuda a:

- reduzir o problema “funciona no meu computador”;
- repetir o mesmo ambiente em desenvolvimento, testes e produção;
- separar serviços diferentes;
- facilitar a instalação de ferramentas como Java e PostgreSQL;
- iniciar e parar aplicações de forma padronizada;
- publicar aplicações em uma [[VPS]] ou em serviços de nuvem.

Docker não é uma máquina virtual completa. Um container compartilha o kernel do sistema operacional e costuma ser mais leve e rápido de iniciar, enquanto uma máquina virtual normalmente inclui um sistema operacional inteiro.

## Conceitos principais

- **Imagem (*image*):** arquivo imutável usado como modelo para criar containers;
- **[[Container]]:** uma imagem em execução;
- **Dockerfile:** arquivo com instruções para construir uma imagem;
- **Registry:** serviço que armazena e distribui imagens, como o Docker Hub ou o Amazon ECR;
- **Volume:** espaço usado para guardar dados que precisam sobreviver à recriação do container;
- **Rede (*network*):** mecanismo que permite que containers se comuniquem;
- **Tag:** rótulo de uma imagem, geralmente usado para identificar uma versão.

Uma imagem pode ser comparada a uma planta de casa. Cada container é uma casa construída a partir dessa planta e que pode ser iniciada, parada ou recriada.

## Dockerfile

Um `Dockerfile` descreve como criar uma imagem. Exemplo para uma aplicação Java empacotada em `target/app.jar`:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app
COPY target/app.jar app.jar

USER 10001
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Esse arquivo:

- usa uma imagem com o ambiente de execução Java 17;
- cria e entra na pasta `/app`;
- copia o pacote gerado pelo Maven;
- troca para um usuário sem privilégios de administrador;
- documenta que a aplicação usa a porta `8080`;
- inicia a aplicação quando o container começa.

O nome do arquivo gerado pelo [[Maven]] pode mudar de acordo com o projeto. Verifique o conteúdo da pasta `target` antes de usar o comando `COPY`.

## Construindo e executando uma imagem

Depois de criar o `Dockerfile`, os comandos básicos são:

```bash
docker build -t estudo-backend:1.0 .
docker run --rm -p 8080:8080 estudo-backend:1.0
```

O que eles fazem:

- `docker build` lê o `Dockerfile` e cria a imagem;
- `-t estudo-backend:1.0` dá nome e versão à imagem;
- `.` informa que o contexto de construção é a pasta atual;
- `docker run` cria e inicia um container;
- `--rm` remove o container quando ele parar;
- `-p 8080:8080` liga a porta do computador à porta do container.

Outros comandos úteis:

```bash
docker ps
docker logs nome-ou-id-do-container
docker stop nome-ou-id-do-container
docker images
docker rm nome-ou-id-do-container
```

- `docker ps` lista containers em execução;
- `docker logs` mostra os registros da aplicação;
- `docker stop` para um container;
- `docker images` lista imagens locais;
- `docker rm` remove um container parado.

## Dados e volumes

O sistema de arquivos de um container pode desaparecer quando o container é removido. Por isso, dados importantes devem ficar em volumes ou em um serviço próprio de armazenamento.

Para um banco como o [[PostgreSQL]], não é seguro depender apenas do conteúdo interno do container. Use um volume, faça backups e teste a restauração.

Um volume pode ser montado assim:

```bash
docker volume create postgres_dados
docker run --mount source=postgres_dados,target=/var/lib/postgresql/data postgres:17
```

O volume `postgres_dados` continua existindo mesmo quando o container é recriado. Isso não substitui backups: se o disco ou o servidor for perdido, o volume também pode ser perdido.

## Redes e vários serviços

Aplicações reais costumam ter vários serviços, como backend, frontend e banco de dados. O Docker permite criar uma rede para que eles se encontrem por nomes de serviço, sem expor tudo diretamente à internet.

O [[Docker Compose]] facilita declarar e iniciar esse conjunto de containers a partir de um arquivo. Ele é especialmente útil para executar a stack localmente.

## Docker na VPS

Em uma [[VPS]], o Docker pode executar os serviços da aplicação de forma isolada. Um fluxo comum é:

1. construir a imagem;
2. enviar a imagem para um registry ou construir diretamente no servidor;
3. iniciar os containers;
4. usar o [[Caddy]] como entrada HTTP e HTTPS;
5. monitorar logs, recursos e saúde dos containers;
6. atualizar as imagens de forma controlada.

O Docker organiza a execução, mas não cuida sozinho de firewall, backups, segredos, atualizações do sistema ou segurança da [[VPS]].

## Docker e AWS

Docker não possui um único equivalente na AWS, porque ele é uma ferramenta de empacotamento e execução. A AWS oferece serviços que podem armazenar e executar imagens Docker:

- **Amazon ECR (*Elastic Container Registry*):** armazena imagens Docker, funcionando como um registry;
- **Amazon ECS (*Elastic Container Service*):** organiza e executa containers;
- **AWS Fargate:** executa tarefas do ECS sem que a equipe administre diretamente os servidores;
- **Amazon EC2:** permite instalar e administrar Docker em máquinas virtuais, oferecendo mais controle.

As principais diferenças são:

- Docker local ou em uma VPS exige que a equipe administre o computador, atualizações e execução;
- ECS e Fargate oferecem mais recursos de orquestração e integração, mas exigem configuração dentro da AWS;
- Fargate reduz a administração de servidores, mas normalmente oferece menos controle direto;
- EC2 com Docker se aproxima mais de uma VPS, pois a equipe ainda administra o sistema operacional;
- ECR guarda imagens, mas não executa containers sozinho.

## Boas práticas

- usar imagens base pequenas, confiáveis e com versão definida, evitando mudanças inesperadas;
- usar builds em múltiplas etapas (*multi-stage builds*) para não levar ferramentas desnecessárias à imagem final;
- executar o processo como usuário sem privilégios, reduzindo o impacto de uma invasão;
- criar um `.dockerignore` para não enviar `.git`, `target`, segredos e arquivos temporários ao contexto de build;
- nunca gravar senhas, tokens ou chaves privadas dentro da imagem;
- usar variáveis de ambiente ou um gerenciador de segredos para configurações sensíveis;
- não usar a tag `latest` em produção, preferindo versões explícitas ou digests;
- atualizar e verificar imagens contra vulnerabilidades;
- expor somente as portas necessárias;
- usar volumes e backups para dados importantes;
- definir limites de CPU e memória, impedindo que um container consuma todo o servidor;
- enviar logs para a saída padrão e centralizá-los no ambiente de execução;
- incluir verificações de saúde (*health checks*) para detectar containers que não estão funcionando corretamente;
- recriar containers de forma controlada, em vez de alterar manualmente o conteúdo interno.

## Resumo

> **Docker é uma ferramenta que empacota aplicações em imagens e as executa em containers isolados e reproduzíveis.**

Ele facilita levar a mesma aplicação do computador de desenvolvimento para uma [[VPS]] ou para serviços de containers na AWS.
