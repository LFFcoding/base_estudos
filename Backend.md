# Backend

**Backend** é a parte de uma aplicação que funciona “por trás” da tela. Ele também é chamado de camada do servidor (*server-side*). O usuário normalmente não vê o backend, mas ele é responsável por processar [[Requisição|requisições]] (*requests*), aplicar regras de negócio (*business rules*) e cuidar dos dados (*data*).

Uma forma simples de imaginar isso é pensar em um restaurante:

- o [[Frontend]] é o salão e o cardápio que o cliente vê;
- o [[Backend]] é a cozinha, que recebe os pedidos e prepara as respostas;
- o [[Banco de dados]] (*database*) é a despensa onde os ingredientes ficam guardados.

## O que o backend faz?

O backend pode:

- receber dados enviados pelo usuário;
- verificar se esses dados são válidos, fazendo uma validação (*validation*);
- aplicar as regras de negócio (*business rules*);
- consultar, criar, alterar ou excluir informações;
- controlar a autenticação (*authentication*) e a autorização (*authorization*) de cada usuário;
- devolver uma resposta (*response*) para o [[Frontend]].

Por exemplo, quando uma pessoa faz login, o backend recebe o usuário e a senha, consulta o banco de dados, verifica se os dados estão corretos e informa se o acesso foi autorizado. Esse processo é uma forma de autenticação (*authentication*).

## [[Frontend]] e [[Backend]] trabalhando juntos

O [[Frontend]] mostra uma tela com um formulário. Quando o usuário preenche esse formulário, o [[Frontend]] envia os dados para o [[Backend]]. O [[Backend]] processa o pedido e devolve uma resposta, como “cadastro realizado” ou “senha incorreta”. Nessa comunicação, o frontend atua como cliente (*client*) e o backend como servidor (*server*).

Essa conversa normalmente acontece por meio de uma [[API]] (*Application Programming Interface*, ou interface de programação de aplicações). A API funciona como um garçom: leva o pedido do frontend até o backend e traz a resposta de volta.

## Backend na stack deste projeto

Neste projeto, o backend será desenvolvido com:

- [[Java|Java 17]], uma [[Linguagem de programação|linguagem de programação]];
- [[Quarkus]], o [[Framework]] que ajuda a construir a aplicação;
- [[Maven]], a ferramenta que organiza o projeto e suas dependências (*dependencies*).

Existem outras opções de backend, como o [[NestJS]], que usa [[Node.js]] e TypeScript. A escolha depende da linguagem, da equipe, dos requisitos e das bibliotecas que o projeto precisa.

O backend também se conecta ao [[PostgreSQL]] para guardar e consultar informações.

## Boas práticas no backend

- validar os dados recebidos antes de processá-los, evitando informações inválidas;
- separar as regras de negócio da comunicação com a tela, facilitando testes e manutenção;
- nunca guardar senhas em texto puro, protegendo os usuários mesmo se os dados forem acessados indevidamente;
- retornar mensagens de erro claras, ajudando o frontend e as pessoas desenvolvedoras a entenderem o problema;
- registrar eventos importantes em logs, criando um histórico útil para investigar falhas;
- testar as regras principais, reduzindo o risco de quebrar funcionalidades existentes.

## Resumo

> **Backend é a parte da aplicação que processa os pedidos, aplica as regras e cuida dos dados.**

Se o frontend é aquilo que o cliente vê em um restaurante, o backend é a cozinha que faz o trabalho acontecer.
