# Docker Compose

**Docker Compose** é uma ferramenta para definir e executar vários containers usando um arquivo de configuração, normalmente chamado `compose.yaml` ou `docker-compose.yml`.

Enquanto o [[Docker]] trabalha com containers individuais, o Docker Compose ajuda a organizar um conjunto de serviços que precisam funcionar juntos.

Uma analogia é organizar uma pequena cidade:

- cada serviço é um prédio com uma função;
- o arquivo Compose é o mapa da cidade;
- a rede permite que os prédios conversem;
- os volumes são os depósitos que mantêm os dados;
- um único comando pode iniciar ou parar a cidade inteira.

Nesta [[Stack]], o Compose pode iniciar o [[Backend]], o [[PostgreSQL]] e outros serviços necessários para estudar e executar a aplicação localmente.

## O arquivo Compose

Um arquivo Compose descreve serviços, imagens, portas, volumes, redes e configurações. Exemplo com um backend Java e PostgreSQL:

```yaml
services:
  backend:
    build: ./backend
    environment:
      DB_URL: jdbc:postgresql://database:5432/estudo
      DB_USER: app
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      database:
        condition: service_healthy
    ports:
      - "8080:8080"

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: estudo
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_dados:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d estudo"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_dados:
```

Nesse exemplo:

- `backend` e `database` são serviços;
- `build: ./backend` diz para construir a imagem usando o `Dockerfile` dessa pasta;
- `image: postgres:17` usa uma imagem pronta do PostgreSQL;
- `database` é o nome usado pelo backend para encontrar o banco na rede interna;
- `depends_on` indica que o backend depende do banco;
- `condition: service_healthy` espera o health check do banco passar;
- `ports` publica a porta `8080` do backend no computador;
- `volumes` mantém os dados do PostgreSQL fora do ciclo de vida do container;
- `${DB_PASSWORD}` lê a senha de uma variável de ambiente.

O exemplo usa o nome `database` em vez de `localhost`. Dentro da rede do Compose, `localhost` seria o próprio container do backend, não o container do PostgreSQL.

## Arquivo `.env`

O Compose pode ler variáveis de um arquivo `.env`:

```dotenv
DB_PASSWORD=troque-esta-senha-localmente
```

O `.env` pode facilitar o estudo local, mas não deve conter segredos reais em um repositório. Adicione-o ao `.gitignore` e mantenha um `.env.example` com nomes de variáveis e valores fictícios:

```dotenv
DB_PASSWORD=defina-uma-senha-local
```

Em produção, prefira um gerenciador de segredos ou variáveis fornecidas pelo ambiente de execução.

## Comandos principais

```bash
docker compose config
docker compose up --build
docker compose up -d
docker compose ps
docker compose logs -f backend
docker compose exec database psql -U app -d estudo
docker compose stop
docker compose down
```

Esses comandos fazem o seguinte:

- `config` valida e mostra a configuração final;
- `up --build` constrói as imagens e inicia os serviços;
- `up -d` inicia os serviços em segundo plano;
- `ps` mostra o estado dos serviços;
- `logs -f backend` acompanha os logs do backend;
- `exec database psql ...` abre o cliente do PostgreSQL dentro do container do banco;
- `stop` para os containers sem removê-los;
- `down` para e remove containers e redes criados pelo Compose.

O comando `docker compose down -v` também remove os volumes declarados. Isso pode apagar os dados locais do banco, portanto só use essa opção quando tiver certeza de que os dados podem ser descartados.

## Ciclo de desenvolvimento

Um fluxo local comum é:

1. editar o código do [[Backend]] ou do [[Frontend]];
2. validar o arquivo com `docker compose config`;
3. executar `docker compose up --build`;
4. acompanhar os logs;
5. testar a aplicação;
6. parar os serviços com `docker compose down` quando terminar.

O Compose facilita que várias pessoas usem serviços com configurações parecidas, reduzindo diferenças entre computadores.

## Volumes

Containers podem ser recriados. Se os dados importantes ficarem apenas dentro do container, eles podem desaparecer quando o container for removido.

Volumes são usados para dados que precisam continuar existindo, como os arquivos do PostgreSQL. Porém, volume não é sinônimo de backup: uma falha no disco ou no servidor pode destruir o volume. Faça cópias de segurança em outro local e teste a restauração.

## Redes

Por padrão, o Compose cria uma rede para os serviços do projeto. Os serviços conseguem se encontrar pelo nome definido no arquivo.

Por segurança, publique para o computador somente as portas que realmente precisam ser acessadas. O PostgreSQL, por exemplo, normalmente deve ficar acessível apenas ao backend, sem uma porta pública para toda a internet.

## Compose em produção

Docker Compose é excelente para desenvolvimento local, testes e aplicações pequenas. Em produção, ele pode ser suficiente para alguns cenários, mas não oferece sozinho recursos completos de alta disponibilidade, escalabilidade automática e distribuição entre vários servidores.

Em uma [[VPS]], o Compose pode iniciar a aplicação, o banco e o [[Caddy]]. Nesse caso, a equipe ainda precisa cuidar do sistema operacional, firewall, backups, atualizações, monitoramento e recuperação de falhas.

## Equivalente na AWS

Não existe um equivalente perfeito ao Docker Compose na AWS. O serviço mais próximo para executar vários containers é o **Amazon ECS** (*Elastic Container Service*), que pode usar o **AWS Fargate** para executar os containers sem administrar diretamente os servidores.

As principais diferenças são:

- Compose usa um arquivo simples e costuma ser executado em um computador ou VPS;
- ECS usa conceitos como task definitions, services, clusters, IAM e redes da AWS;
- Fargate reduz a administração de servidores, mas exige configurar recursos da AWS e pode ter cobrança mais detalhada;
- no Compose, a equipe organiza diretamente os containers;
- no ECS, a AWS ajuda a manter tarefas, distribuir serviços e integrar balanceadores, logs e permissões;
- o **Amazon ECR** pode armazenar as imagens que o ECS ou Fargate executará.

Uma aplicação pode começar com Docker Compose localmente e depois ser adaptada para ECS, mas o arquivo Compose não é automaticamente uma configuração completa de produção na AWS.

## Boas práticas

- validar o arquivo com `docker compose config` antes de iniciar os serviços;
- fixar versões das imagens, evitando usar `latest` em ambientes importantes;
- manter senhas e tokens fora do arquivo Compose e do [[GitHub]];
- usar `.env.example` para documentar variáveis sem expor segredos;
- definir health checks para saber quando um serviço está realmente pronto;
- usar `depends_on` junto com health checks, lembrando que iniciar um container não significa que a aplicação já está pronta;
- usar volumes para dados persistentes e backups fora dos containers;
- expor somente as portas necessárias;
- limitar CPU e memória em ambientes compartilhados;
- separar configurações de desenvolvimento e produção;
- acompanhar logs e não esconder erros com comandos executados em segundo plano;
- manter imagens atualizadas e verificadas contra vulnerabilidades;
- evitar colocar muitos serviços diferentes no mesmo container, mantendo responsabilidades claras.

## Resumo

> **Docker Compose é uma forma simples de definir e executar vários containers como um conjunto.**

Ele funciona como um mapa da aplicação: descreve quais serviços existem, como eles se conectam, quais dados persistem e como tudo deve ser iniciado.
