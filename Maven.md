# Maven

**Maven** é uma ferramenta usada em projetos Java para organizar o código, baixar bibliotecas, compilar a aplicação, executar testes e gerar o arquivo que será publicado.

O comando principal do Maven é `mvn`.

Uma analogia é pensar em uma lista de compras e em um organizador de obra:

- o projeto informa quais materiais precisa;
- o Maven baixa as bibliotecas corretas;
- ele segue etapas conhecidas para compilar e testar;
- no final, prepara um pacote da aplicação.

Nesta [[Stack]], o Maven organiza projetos feitos com [[Java]] e [[Quarkus]].

## O que o Maven faz?

O Maven ajuda a:

- organizar a estrutura do projeto;
- baixar e controlar dependências (*dependencies*);
- compilar código Java;
- executar testes;
- criar pacotes `.jar`;
- executar plugins (*plugins*) com tarefas específicas;
- padronizar comandos para toda a equipe;
- preparar a aplicação para ambientes como [[Docker Compose]] ou uma [[VPS]].

## O arquivo `pom.xml`

Todo projeto Maven possui normalmente um arquivo chamado `pom.xml`. POM significa *Project Object Model*, ou modelo de objeto do projeto.

Esse arquivo descreve o projeto e suas necessidades. Um exemplo simples é:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.exemplo</groupId>
    <artifactId>estudo-java</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.release>17</maven.compiler.release>
    </properties>
</project>
```

Nesse exemplo:

- `groupId` identifica o grupo ou organização do projeto;
- `artifactId` identifica o nome do projeto;
- `version` identifica a versão;
- `maven.compiler.release` informa que o código usa Java 17.

Uma aplicação Quarkus normalmente possui um `pom.xml` maior, com o [[Framework]], extensões e dependências necessárias.

## Dependências

Uma dependência (*dependency*) é uma [[Biblioteca]] ou ferramenta externa que o projeto usa.

Exemplo de uma dependência declarada no `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.11.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

O Maven encontra a dependência em um repositório (*repository*), baixa seus arquivos e os coloca à disposição durante a compilação ou os testes.

O `scope` `test` informa que essa dependência só é necessária para executar testes, não para rodar a aplicação em produção.

## Ciclo de vida

O Maven possui etapas organizadas em um ciclo de vida (*lifecycle*). Algumas fases importantes são:

- `validate`: verifica se o projeto está estruturado corretamente;
- `compile`: compila o código principal;
- `test`: executa os testes;
- `package`: cria o pacote da aplicação;
- `verify`: executa verificações adicionais;
- `install`: instala o pacote no repositório local;
- `clean`: remove arquivos gerados anteriormente.

Quando executamos uma fase, o Maven também executa as fases anteriores necessárias.

Por exemplo, `mvn package` normalmente valida, compila, testa e depois cria o pacote.

## Comandos comuns

```bash
mvn clean
mvn test
mvn package
mvn verify
mvn dependency:tree
mvn quarkus:dev
```

O que cada comando faz:

- `mvn clean`: remove a pasta `target`, que contém arquivos gerados;
- `mvn test`: executa os testes;
- `mvn package`: gera o pacote da aplicação;
- `mvn verify`: executa verificações do projeto;
- `mvn dependency:tree`: mostra as dependências diretas e indiretas;
- `mvn quarkus:dev`: inicia uma aplicação [[Quarkus]] em modo de desenvolvimento.

## Maven Wrapper

O Maven Wrapper permite executar o Maven com uma versão definida pelo próprio projeto, usando arquivos como `mvnw` ou `mvnw.cmd`.

Exemplo no macOS ou Linux:

```bash
./mvnw test
./mvnw package
```

Isso reduz o risco de duas pessoas usarem versões incompatíveis do Maven. Quando o projeto possui Wrapper, prefira `./mvnw` em vez de depender da instalação global da máquina.

## Maven e Quarkus

O [[Quarkus]] usa Maven para:

- baixar extensões;
- compilar o backend;
- executar testes;
- iniciar o modo de desenvolvimento;
- gerar o pacote da aplicação;
- preparar a execução em containers.

Assim, o Maven é uma das ferramentas que conecta o código Java ao processo de construção e publicação do backend.

## Boas práticas

- usar o Maven Wrapper para padronizar a versão da ferramenta;
- definir versões de dependências e plugins, evitando mudanças inesperadas;
- manter dependências atualizadas com cuidado, verificando compatibilidade e segurança;
- usar apenas bibliotecas realmente necessárias, reduzindo complexidade;
- revisar `mvn dependency:tree` para entender dependências indiretas;
- executar testes antes de empacotar ou publicar a aplicação;
- não versionar a pasta `target`, pois ela pode ser recriada pelo Maven;
- separar configurações sensíveis do `pom.xml` e do repositório;
- preferir uma versão estável e suportada do Java, como Java 17;
- investigar avisos e erros do Maven em vez de ignorá-los;
- manter o `pom.xml` organizado, com dependências agrupadas e nomes claros.

## Resumo

> **Maven é uma ferramenta que organiza projetos Java, controla dependências e automatiza a compilação, os testes e a criação de pacotes.**

O arquivo `pom.xml` funciona como a receita do projeto, e os comandos `mvn` executam as etapas dessa receita.
