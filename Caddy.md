# Caddy

**Caddy** é um servidor web (*web server*) e proxy reverso (*reverse proxy*). Ele recebe conexões da internet e encaminha cada requisição para o serviço correto.

Nesta [[Stack]], o Caddy pode ficar na frente do [[Frontend]] e do [[Backend]], recebendo acessos HTTP e HTTPS. Ele também pode servir arquivos estáticos, como [[HTML]], [[CSS]], JavaScript e imagens.

Uma analogia é a entrada de um prédio:

- o Caddy é a recepção;
- a pessoa visitante é quem faz a [[Requisição]];
- a recepção verifica o endereço e encaminha a pessoa para o setor correto;
- os setores internos são os serviços da aplicação;
- o usuário não precisa conhecer os endereços internos de cada serviço.

## O que o Caddy faz?

O Caddy pode:

- servir arquivos de um site;
- encaminhar requisições para um backend;
- terminar conexões HTTPS;
- obter e renovar certificados TLS automaticamente;
- redirecionar HTTP para HTTPS;
- aplicar regras por domínio e caminho;
- comprimir respostas;
- registrar logs de acesso e erros;
- distribuir requisições entre várias instâncias do mesmo serviço.

O Caddy não substitui o [[Backend]]. Ele normalmente fica na frente do backend e encaminha os pedidos para ele.

## Proxy reverso

Um proxy reverso recebe o pedido do cliente e conversa com o serviço interno em nome do cliente.

Por exemplo:

1. a pessoa acessa `https://exemplo.com`;
2. o Caddy recebe a conexão;
3. o Caddy verifica a regra do domínio;
4. o Caddy encaminha a requisição para `backend:8080`;
5. o backend processa o pedido;
6. o Caddy devolve a resposta ao usuário.

O usuário vê apenas `exemplo.com`. O endereço e a porta internos do backend não precisam ser expostos diretamente.

## Caddyfile

O arquivo de configuração mais comum do Caddy é chamado `Caddyfile`. Um exemplo simples é:

```caddyfile
api.exemplo.com {
    reverse_proxy backend:8080
}
```

Esse arquivo diz: “quando alguém acessar `api.exemplo.com`, encaminhe a requisição para o serviço `backend` na porta `8080`”.

O nome `backend` pode ser o nome do serviço em uma rede do [[Docker Compose]]. Dentro dessa rede, não use `localhost` para apontar para outro container: `localhost` significa o próprio container do Caddy.

## Servindo arquivos estáticos

Para servir os arquivos de um frontend já compilado:

```caddyfile
exemplo.com {
    root * /srv/frontend
    encode gzip
    file_server
}
```

Nesse exemplo:

- `root` define a pasta dos arquivos;
- `encode gzip` permite comprimir respostas;
- `file_server` habilita o servidor de arquivos.

Também é possível separar o site e a API por domínios:

```caddyfile
exemplo.com {
    root * /srv/frontend
    file_server
}

api.exemplo.com {
    reverse_proxy backend:8080
}
```

## HTTPS automático

Quando um domínio real aponta para o servidor e as portas `80` e `443` estão acessíveis, o Caddy pode solicitar e renovar certificados TLS automaticamente.

HTTPS protege os dados no caminho entre o navegador e o servidor. Para isso funcionar, é necessário:

- registrar um domínio;
- apontar o DNS para o endereço do servidor;
- liberar as portas `80` e `443` no firewall;
- garantir que o Caddy seja acessível nessas portas;
- usar um domínio válido, não apenas `localhost`.

O certificado protege a conexão até o Caddy. A comunicação entre Caddy e backend também deve ser protegida quando atravessar redes não confiáveis.

## Caddy com Docker Compose

Um serviço Caddy pode ser declarado assim:

```yaml
services:
  caddy:
    image: caddy:2-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
    depends_on:
      - backend

  backend:
    build: ./backend

volumes:
  caddy_data:
  caddy_config:
```

Os volumes `/data` e `/config` são importantes para preservar certificados, dados e configurações do Caddy quando o container for recriado.

Em produção, fixe uma versão validada da imagem em vez de depender somente de uma tag genérica como `2-alpine`.

## Comandos úteis

```bash
docker compose up -d caddy
docker compose logs -f caddy
docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
docker compose restart caddy
```

- `up -d caddy` inicia o serviço em segundo plano;
- `logs -f` acompanha os logs;
- `caddy validate` verifica a configuração antes de usá-la;
- `restart` reinicia o serviço depois de uma mudança.

Sempre valide o `Caddyfile` antes de reiniciar. Uma configuração inválida pode deixar o site indisponível.

## Caddy em uma VPS

Em uma [[VPS]], o Caddy pode ser o ponto público da aplicação:

1. o firewall libera apenas as portas necessárias;
2. o Caddy recebe HTTP e HTTPS;
3. o Caddy serve o frontend ou encaminha para o backend;
4. o PostgreSQL permanece em uma rede interna;
5. os serviços internos não precisam expor suas portas para a internet.

O Caddy simplifica a entrada web, mas a equipe ainda precisa cuidar do sistema operacional, firewall, atualizações, backups, logs e monitoramento da VPS.

## Equivalentes na AWS

Não existe um equivalente único ao Caddy na AWS, porque ele combina servidor web, proxy reverso e gerenciamento de certificados. Uma composição próxima seria:

- **Application Load Balancer (ALB):** recebe tráfego e encaminha requisições para serviços;
- **AWS Certificate Manager (ACM):** fornece e gerencia certificados TLS para serviços compatíveis;
- **Amazon CloudFront:** pode atuar como CDN e camada de entrada global;
- **Amazon ECS ou Fargate:** executa os containers do backend.

As principais diferenças são:

- Caddy é um programa que a equipe instala e configura em uma VPS ou container;
- ALB e ACM são serviços gerenciados, reduzindo a administração de servidores;
- Caddy pode servir arquivos diretamente, enquanto na AWS arquivos estáticos costumam ser colocados em S3 e distribuídos pelo CloudFront;
- a AWS oferece mais integração e escala, mas pode exigir mais configurações e gerar cobrança por vários serviços;
- Caddy é simples para uma aplicação em uma VPS, enquanto ALB, ACM e CloudFront são úteis quando o projeto precisa usar vários serviços AWS.

## Boas práticas

- usar HTTPS em produção e redirecionar HTTP para HTTPS;
- validar o `Caddyfile` antes de reiniciar o serviço;
- não expor diretamente portas internas do backend e do banco;
- manter certificados e configurações persistentes em volumes ou armazenamento seguro;
- guardar segredos fora do `Caddyfile` e do repositório do [[GitHub]];
- fixar e atualizar a imagem do Caddy de forma controlada;
- acompanhar logs de acesso e erro, sem registrar tokens ou senhas;
- configurar limites e regras adequadas contra abuso;
- manter o Caddy e o sistema operacional atualizados;
- testar a renovação de certificados e a restauração das configurações;
- configurar health checks e respostas de erro claras;
- separar regras do frontend e da API para facilitar manutenção.

## Resumo

> **Caddy é um servidor web e proxy reverso que recebe requisições, serve arquivos, encaminha pedidos e pode automatizar o HTTPS.**

Ele funciona como a porta de entrada da aplicação, escondendo os serviços internos e encaminhando cada pedido para o lugar correto.
