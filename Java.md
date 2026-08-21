# Java

**Java** é uma [[Linguagem de programação|linguagem de programação]] e também uma plataforma usada para criar diferentes tipos de sistemas.

Nesta [[Stack]], o Java 17 será usado principalmente para construir o [[Backend]]. Ele trabalhará junto com o [[Quarkus]], o [[Maven]] e o [[PostgreSQL]].

Uma analogia é pensar em uma receita de bolo:

- o código Java é a receita escrita pela pessoa desenvolvedora;
- o compilador transforma a receita em um formato intermediário;
- a JVM funciona como a cozinha que entende esse formato e executa o programa;
- o sistema operacional é o espaço onde essa cozinha funciona.

## Por que Java pode funcionar em diferentes sistemas?

O código Java costuma ser compilado para **bytecode**, um formato intermediário. Esse bytecode é executado pela JVM (*Java Virtual Machine*, ou Máquina Virtual Java).

Por isso, o mesmo programa pode funcionar em diferentes sistemas operacionais quando existe uma JVM compatível. Essa ideia é conhecida pela frase “escreva uma vez, execute em qualquer lugar” (*write once, run anywhere*), embora ainda seja necessário testar o programa no ambiente real.

## JDK, JVM e JRE

Esses nomes representam partes diferentes da plataforma Java:

- **JVM (*Java Virtual Machine*):** executa o bytecode Java;
- **JDK (*Java Development Kit*):** conjunto usado para desenvolver Java. Inclui o compilador, a JVM e outras ferramentas;
- **JRE (*Java Runtime Environment*):** nome usado para representar o ambiente necessário para executar aplicações Java.

Para programar, normalmente instalamos o JDK. Ele fornece comandos como `javac`, que compila o código, e `java`, que executa a aplicação.

## Primeiro exemplo

Crie um arquivo chamado `Saudacao.java`:

```java
public class Saudacao {
    public static void main(String[] args) {
        String nome = "Ana";
        System.out.println("Olá, " + nome + "!");
    }
}
```

Depois, compile e execute:

```bash
javac Saudacao.java
java Saudacao
```

O comando `javac` transforma `Saudacao.java` em bytecode. O comando `java` pede para a JVM executar a classe `Saudacao`.

## O que significa cada parte?

- `public class Saudacao`: cria uma classe pública chamada `Saudacao`;
- `main`: é o ponto inicial da aplicação;
- `String nome`: cria uma variável de texto;
- `System.out.println`: escreve uma mensagem no [[Terminal]];
- `"Olá, " + nome`: junta textos para formar a mensagem final.

## Java no backend

No [[Backend]], Java pode ser usado para:

- criar APIs;
- aplicar regras de negócio;
- validar dados recebidos em uma [[Requisição]];
- acessar o [[PostgreSQL]];
- controlar autenticação e autorização;
- executar tarefas em segundo plano;
- integrar diferentes sistemas.

O Java puro fornece a linguagem e sua plataforma. O [[Quarkus]] é um [[Framework]] que oferece uma estrutura pronta para criar aplicações backend. O [[Maven]] organiza a compilação, as dependências e outras tarefas do projeto.

## Java não é [[JavaScript]]

Apesar dos nomes parecidos, [[Java]] e [[JavaScript]] são linguagens diferentes:

- Java é muito usado em backend, aplicações corporativas, serviços e sistemas Android antigos;
- [[JavaScript]] é muito usado no navegador e também pode ser usado no servidor;
- os dois possuem sintaxes e ferramentas próprias.

Aprender uma não significa automaticamente saber a outra.

## Boas práticas em Java

- usar uma versão com suporte de longo prazo (*Long-Term Support* ou LTS), como Java 17;
- escolher nomes claros para classes, métodos e variáveis, tornando o código mais fácil de ler;
- manter cada classe com uma responsabilidade bem definida, facilitando testes e manutenção;
- validar dados recebidos antes de usá-los, evitando erros e informações inválidas;
- tratar exceções de forma adequada, sem esconder problemas importantes;
- escrever testes para as regras principais, reduzindo o risco de quebrar funcionalidades;
- não guardar senhas, tokens ou chaves privadas no código;
- manter dependências como bibliotecas e frameworks atualizadas com cuidado;
- usar formatação e análise automática de código para manter um padrão no projeto;
- preferir código simples antes de criar abstrações desnecessárias.

## Resumo

> **Java é uma linguagem e uma plataforma de programação que permite criar aplicações executadas pela JVM.**

No projeto de estudos, Java 17 será a base do backend, enquanto Quarkus ajudará a organizar a aplicação e Maven cuidará da construção e das dependências.
