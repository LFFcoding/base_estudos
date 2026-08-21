# VPS

**VPS** significa *Virtual Private Server*, ou **Servidor Virtual Privado**. É um servidor virtual que funciona dentro de um servidor físico maior e fica conectado à internet.

Uma analogia é imaginar um prédio:

- o servidor físico é o prédio inteiro;
- cada VPS é como um apartamento separado;
- cada apartamento tem seus próprios recursos, como memória, espaço e capacidade de processamento;
- a empresa de hospedagem cuida do prédio e do computador físico;
- a pessoa responsável pela VPS cuida do seu apartamento e do que instala dentro dele.

Uma VPS pode hospedar os serviços da [[Stack]], como o [[Backend]], o [[PostgreSQL]], o [[Docker Compose]] e o [[Caddy]].

## O que uma VPS oferece?

Uma VPS normalmente oferece:

- um sistema operacional, como Linux;
- uma quantidade definida de memória RAM;
- uma quantidade de processamento, geralmente em vCPUs;
- espaço para armazenar arquivos;
- um endereço IP público;
- acesso administrativo, normalmente por [[SSH]] (*Secure Shell*).

Com esse acesso, é possível instalar programas, configurar usuários, publicar aplicações e controlar os serviços do servidor.

## VPS não é um computador mágico

Contratar uma VPS não deixa a aplicação pronta automaticamente. A empresa fornece o servidor virtual, mas ainda é necessário:

- atualizar e proteger o sistema operacional;
- instalar as ferramentas necessárias;
- configurar o [[Docker Compose]] e o [[Caddy]];
- publicar o [[Backend]] e o [[Frontend]];
- configurar o [[PostgreSQL]];
- criar backups e monitorar os serviços.

Por isso, uma VPS costuma ser chamada de serviço **IaaS** (*Infrastructure as a Service*, ou infraestrutura como serviço): o provedor entrega a infraestrutura básica, e a equipe administra boa parte do restante.

## Como acessar uma VPS?

O acesso mais comum em servidores Linux é feito por SSH:

```bash
ssh usuario@203.0.113.10
```

Esse comando significa: “conecte-se ao servidor usando o usuário `usuario` e o endereço `203.0.113.10`”. O endereço usado acima é apenas um exemplo reservado para documentação; não é um servidor real.

Depois de entrar, um projeto que usa Docker Compose pode ser iniciado com:

```bash
docker compose up -d
```

O parâmetro `-d` faz os serviços serem executados em segundo plano (*detached mode*), permitindo fechar a conexão SSH sem interromper os containers.

## VPS e publicação de uma aplicação

Um fluxo simples de publicação pode ser:

1. contratar uma VPS;
2. configurar o sistema operacional e um usuário administrativo seguro;
3. instalar Docker e Docker Compose;
4. configurar o Caddy para receber conexões HTTP e HTTPS;
5. baixar o código do [[GitHub]];
6. configurar variáveis de ambiente e segredos fora do código;
7. iniciar os serviços com Docker Compose;
8. configurar backups, logs e monitoramento.

O Caddy pode receber uma [[Requisição]] da internet e encaminhá-la para o serviço correto. Dessa forma, a VPS funciona como o local onde a aplicação fica executando e disponível para os usuários.

## VPS comparada com outras opções

- **Hospedagem compartilhada:** vários clientes dividem um ambiente mais limitado. É simples, mas oferece menos controle.
- **VPS:** oferece um ambiente virtual separado e mais controle, normalmente por um preço mensal previsível.
- **Servidor dedicado:** uma máquina física inteira fica reservada para um cliente. Oferece mais recursos, mas costuma custar mais e exigir mais administração.
- **Serviço gerenciado:** o provedor administra uma parte maior da operação. Facilita o trabalho, mas pode reduzir o controle e aumentar o custo.

## Equivalente na AWS

Na AWS, o serviço mais parecido com uma VPS é o **Amazon EC2** (*Elastic Compute Cloud*). Ele permite criar uma máquina virtual, escolher recursos, instalar um sistema operacional e administrar o servidor.

As principais diferenças são:

- uma VPS tradicional costuma oferecer planos mensais mais simples e previsíveis;
- o EC2 permite escolher muitos tipos de máquina e integrar serviços da AWS, mas a configuração e a cobrança podem ser mais complexas;
- na VPS, o provedor normalmente reúne recursos e suporte em um pacote;
- no EC2, vários itens podem ser cobrados separadamente, como armazenamento, tráfego e endereços IP;
- os dois modelos deixam a administração do sistema operacional e da aplicação principalmente com a equipe responsável.

O EC2 é equivalente na ideia, mas não é uma cópia exata de toda VPS. A escolha depende do quanto o projeto precisa de simplicidade, flexibilidade, integração e escala.

## Boas práticas de segurança

- usar chaves SSH em vez de senhas sempre que possível, dificultando tentativas de acesso;
- criar um usuário comum e usar `sudo` apenas quando necessário, evitando trabalhar sempre como `root`;
- manter o sistema operacional, Docker e demais ferramentas atualizados, corrigindo vulnerabilidades;
- configurar um firewall para liberar somente as portas necessárias, normalmente SSH, HTTP e HTTPS;
- não deixar o [[PostgreSQL]] exposto diretamente à internet, permitindo acesso apenas aos serviços que precisam dele;
- usar HTTPS e certificados [[TLS]] válidos no Caddy, protegendo dados entre o usuário e o servidor;
- guardar senhas, tokens e chaves em variáveis de ambiente ou gerenciadores de segredos, nunca no repositório do [[GitHub]];
- fazer backups automáticos e testar a restauração, porque um backup que nunca foi testado pode não funcionar;
- manter cópias dos backups fora da VPS, protegendo os dados contra falha ou perda do servidor;
- acompanhar logs, uso de CPU, memória e espaço em disco, percebendo problemas antes que a aplicação pare.

## O que fica sob sua responsabilidade?

Em uma VPS, o provedor geralmente cuida do servidor físico, da virtualização e da rede básica. A equipe responsável pelo projeto normalmente cuida de:

- sistema operacional;
- usuários e permissões;
- firewall e atualizações;
- aplicações e containers;
- banco de dados e backups;
- certificados, segredos e monitoramento.

Essa divisão é importante: se uma aplicação estiver vulnerável ou uma senha for exposta, contratar uma VPS não transfere automaticamente essa responsabilidade para o provedor.

## Resumo

> **VPS é um servidor virtual conectado à internet, com recursos reservados e controle suficiente para hospedar aplicações.**

Ela oferece mais liberdade que uma hospedagem compartilhada, mas também exige que a equipe cuide da configuração, segurança, atualizações e backups.
