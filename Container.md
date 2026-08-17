# Container

Um **container** é um processo isolado que executa uma aplicação e seus arquivos necessários dentro de um sistema operacional.

Containers são criados a partir de [[Docker|imagens Docker]]. A imagem é o modelo parado; o container é esse modelo funcionando.

Uma analogia é uma caixa de transporte:

- a imagem é o molde da caixa com a mercadoria preparada;
- o container é a caixa em uso;
- a aplicação é a mercadoria;
- as portas são as aberturas por onde a caixa se comunica;
- o volume é um compartimento externo usado para guardar algo que não pode ser perdido.

## Container não é máquina virtual

Uma máquina virtual geralmente inclui um sistema operacional inteiro. Um container compartilha o kernel do sistema operacional do computador ou servidor e isola principalmente processos, arquivos e rede.

Por isso, containers costumam:

- iniciar mais rapidamente;
- usar menos recursos;
- ser fáceis de criar e remover;
- funcionar de maneira parecida em ambientes diferentes.

Esse isolamento não é uma proteção absoluta. Um container mal configurado pode expor o servidor, por isso a segurança ainda precisa ser planejada.

## Imagem e container

Uma imagem pode ser usada para criar vários containers. Por exemplo, uma imagem do PostgreSQL pode gerar um container para desenvolvimento e outro para testes.

Alterar um container manualmente não altera a imagem original. Se a mudança for importante, registre-a em um `Dockerfile` ou em uma configuração do [[Docker Compose]], para que o ambiente possa ser recriado.

## Ciclo de vida

Um container costuma passar por estas etapas:

1. **criado (*created*):** sua estrutura foi preparada, mas ele ainda não iniciou;
2. **executando (*running*):** o processo principal está funcionando;
3. **parado (*stopped*):** o processo foi encerrado, mas o container ainda existe;
4. **reiniciado (*restarted*):** o processo foi iniciado novamente;
5. **removido (*removed*):** o container deixou de existir.

O container deve ser visto como algo descartável e recriável. Dados importantes não devem depender apenas do sistema de arquivos interno dele.

## Criando um container

O comando abaixo executa um servidor web de exemplo:

```bash
docker run --name exemplo-web -d -p 8080:80 nginx:1.27
```

Esse comando:

- cria um container chamado `exemplo-web`;
- usa a imagem `nginx` na versão `1.27`;
- executa o processo em segundo plano com `-d`;
- conecta a porta `8080` do computador à porta `80` do container.

Depois, `http://localhost:8080` pode ser aberto no navegador para testar o servidor.

## Inspecionando e controlando

```bash
docker ps
docker logs exemplo-web
docker exec -it exemplo-web sh
docker stop exemplo-web
docker rm exemplo-web
```

- `docker ps` mostra containers em execução;
- `docker logs` mostra a saída do processo principal;
- `docker exec -it` abre um shell dentro do container;
- `docker stop` para o container;
- `docker rm` remove um container parado.

Entrar no container com `docker exec` é útil para investigar problemas, mas não deve ser a forma principal de configurar a aplicação. A configuração correta deve estar no código, na imagem ou nos arquivos de implantação.

## Portas

Uma aplicação dentro do container pode escutar em uma porta, mas essa porta não fica automaticamente disponível fora dele.

No exemplo `-p 8080:80`:

- `8080` é a porta do computador ou servidor;
- `80` é a porta usada pelo processo dentro do container.

Em produção, publique somente as portas necessárias. Um banco de dados como o [[PostgreSQL]] normalmente deve ser acessado apenas por outros serviços da rede interna, não diretamente pela internet.

## Volumes e dados persistentes

O container pode ser removido e recriado. Para manter dados, use um volume:

```bash
docker volume create dados-postgres
docker run --name banco-estudo \
    --mount source=dados-postgres,target=/var/lib/postgresql/data \
    postgres:17
```

O volume continua separado do ciclo de vida do container. Mesmo assim, ele não substitui backups: uma falha no disco ou no servidor pode apagar o volume.

## Containers conversando

Em uma aplicação real, o [[Backend]], o [[Frontend]] e o PostgreSQL podem ficar em containers diferentes. Eles se comunicam por uma rede Docker.

O [[Docker Compose]] facilita declarar os serviços, a rede, os volumes e as variáveis de ambiente em um único arquivo. Em vez de usar `localhost`, um serviço normalmente acessa outro pelo nome do serviço, como `database:5432`.

## Container na VPS

Em uma [[VPS]], containers podem organizar os serviços da aplicação e facilitar atualizações. Porém, a equipe continua responsável por:

- manter o Docker e o sistema operacional atualizados;
- configurar firewall e portas;
- proteger os segredos;
- criar backups;
- acompanhar logs e consumo de recursos;
- reiniciar ou substituir containers com segurança.

O container facilita a execução, mas não substitui a administração do servidor.

## Equivalentes na AWS

Na AWS, containers podem ser executados em diferentes serviços:

- **Amazon ECS:** gerencia serviços e tarefas baseadas em containers;
- **AWS Fargate:** executa containers sem que a equipe administre diretamente as máquinas;
- **Amazon EKS:** executa containers usando Kubernetes;
- **Amazon EC2:** permite instalar Docker e administrar os containers diretamente na máquina virtual;
- **Amazon ECR:** armazena as imagens usadas por esses serviços.

ECS e Fargate oferecem mais automação e integração com a AWS, mas exigem configuração de rede, permissões, logs e serviços. Executar Docker em EC2 se aproxima mais do uso de uma VPS, pois a equipe mantém maior controle e também assume mais responsabilidades.

## Boas práticas de segurança

- usar imagens pequenas, confiáveis e com versões fixadas;
- executar o processo como usuário sem privilégios, evitando `root` quando não for necessário;
- não usar `--privileged` sem entender exatamente o risco;
- não montar o socket do Docker dentro do container, pois isso pode permitir controlar o servidor;
- não guardar senhas, tokens ou chaves privadas no sistema de arquivos da imagem;
- usar variáveis de ambiente ou um gerenciador de segredos;
- limitar CPU e memória para evitar que um container consuma todo o servidor;
- usar sistemas de arquivos somente leitura quando a aplicação permitir;
- remover capacidades do Linux que não forem necessárias;
- verificar imagens contra vulnerabilidades antes de executá-las;
- não expor portas de banco de dados publicamente;
- manter dados fora do container, usando volumes e backups;
- registrar logs na saída padrão para que o ambiente consiga coletá-los;
- definir health checks e uma política de reinício adequada.

## Resumo

> **Container é um processo isolado que executa uma aplicação de forma leve, reproduzível e controlada.**

Ele é criado a partir de uma imagem, pode conversar com outros containers e deve ser fácil de recriar sem perder dados importantes.
