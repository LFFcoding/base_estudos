# Quarkus

**Quarkus** é um [[Framework]] Java usado para criar aplicações backend, especialmente APIs e serviços que serão executados em containers ou ambientes de nuvem.

Nesta [[Stack]], o Quarkus ajuda a construir o [[Backend]] com [[Java]]. Ele trabalha junto com o [[Maven]], o [[PostgreSQL]] e o [[Docker Compose]].

Uma analogia é pensar em construir uma casa:

- [[Java]] é a linguagem e os materiais básicos;
- o Quarkus é uma planta com estruturas prontas para construir um tipo de casa;
- o [[Maven]] organiza os materiais e as etapas da obra;
- o backend é a casa funcionando, recebendo pedidos e entregando respostas.

## O que o Quarkus facilita?

O Quarkus oferece recursos para:

- criar APIs HTTP;
- receber e responder [[Requisição|requisições]];
- organizar regras de negócio;
- conectar a aplicação ao [[PostgreSQL]];
- validar dados;
- configurar autenticação e autorização;
- executar testes;
- criar aplicações para containers;
- acompanhar a saúde da aplicação;
- usar recarregamento automático durante o desenvolvimento.

O objetivo é fornecer uma base pronta, para que a pessoa desenvolvedora se concentre nas regras do sistema em vez de construir toda a infraestrutura do backend do zero.

## Por que o Quarkus é conhecido por ser rápido?

O Quarkus foi pensado para iniciar rapidamente e consumir menos memória, características úteis em containers e ambientes de nuvem.

Ele pode executar uma aplicação de duas formas principais:

- **modo JVM:** o código roda na Java Virtual Machine, como em outras aplicações Java;
- **modo nativo (*native executable*):** a aplicação é compilada para um executável que pode iniciar ainda mais rápido e usar menos memória em alguns cenários.

O modo nativo pode exigir ajustes e um processo de compilação diferente. Por isso, é importante medir o resultado real antes de escolher essa opção.

## Criando um endpoint

Um endpoint é um endereço que recebe uma [[Requisição]] e devolve uma resposta. Um exemplo simples em Quarkus é:

```java
package com.exemplo;

import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;

@Path("/saudacoes")
public class SaudacaoResource {

    @GET
    public String saudacao() {
        return "Olá, Quarkus!";
    }
}
```

Essa classe cria um endpoint `GET /saudacoes`:

- `@Path` define o caminho da URL;
- `@GET` indica que o método responde a requisições HTTP GET;
- `saudacao` devolve o texto da resposta.

Se a aplicação estiver rodando localmente, uma requisição para `http://localhost:8080/saudacoes` deve retornar `Olá, Quarkus!`.

## Executando em modo de desenvolvimento

Dentro de um projeto Quarkus criado com Maven, o comando mais comum é:

```bash
mvn quarkus:dev
```

Esse comando inicia o modo de desenvolvimento (*development mode*). Enquanto a aplicação está rodando, alterações no código podem ser recarregadas automaticamente, facilitando os testes locais.

O modo de desenvolvimento não deve ser usado como configuração de produção. Em produção, a aplicação deve ser empacotada, configurada com variáveis adequadas e executada em um ambiente controlado.

## Configuração

Configurações como endereço do banco, usuário e ambiente devem ficar fora do código. Um exemplo de configuração pode ser:

```properties
quarkus.datasource.jdbc.url=${DB_URL}
quarkus.datasource.username=${DB_USER}
quarkus.datasource.password=${DB_PASSWORD}
```

O Quarkus lê os valores das variáveis de ambiente `DB_URL`, `DB_USER` e `DB_PASSWORD`. Assim, a mesma aplicação pode ser executada em ambientes diferentes sem alterar o código.

Não coloque senhas reais em arquivos versionados ou no repositório do [[GitHub]].

## Quarkus e Maven

O [[Maven]] organiza o projeto, baixa dependências (*dependencies*), compila o código e executa tarefas do Quarkus.

Alguns comandos comuns dentro do projeto são:

```bash
mvn test
mvn package
mvn quarkus:dev
```

- `mvn test` executa os testes;
- `mvn package` compila e empacota a aplicação;
- `mvn quarkus:dev` inicia o modo de desenvolvimento.

## Quarkus e containers

O Quarkus combina bem com [[Docker Compose]] porque a aplicação pode ser empacotada em um container e executada junto com outros serviços, como o PostgreSQL.

Essa separação é parecida com uma cozinha organizada: cada container cuida de uma função, mas os serviços conversam por uma rede definida no ambiente de execução.

## Boas práticas

- separar as regras de negócio dos recursos HTTP, facilitando testes e mudanças;
- usar configurações por ambiente, evitando alterar o código para cada lugar onde a aplicação roda;
- manter senhas e tokens fora do código e do repositório;
- escrever testes para as regras principais e endpoints importantes;
- validar dados recebidos antes de usá-los;
- definir respostas e erros consistentes na API;
- usar o modo de desenvolvimento somente localmente;
- fixar e atualizar as versões das dependências com cuidado;
- monitorar logs, saúde, memória e tempo de resposta em produção;
- medir startup e consumo antes de escolher o modo nativo;
- empacotar a aplicação de forma reproduzível, facilitando a publicação em containers.

## Resumo

> **Quarkus é um framework Java que facilita a criação de backends rápidos, organizados e adequados para containers e ambientes de nuvem.**

Ele não substitui o Java: usa Java como base e oferece ferramentas e padrões para construir a aplicação com menos trabalho repetitivo.
