# Linguagem de programação

Uma **linguagem de programação** (*programming language*) é uma forma organizada de escrever instruções que um computador consegue executar.

O computador trabalha com sinais elétricos e números, mas as pessoas precisam de uma forma mais compreensível de dar ordens. A linguagem de programação funciona como uma ponte entre a ideia humana e as operações que a máquina realiza.

Uma analogia é uma receita de cozinha:

- a pessoa escreve os passos usando regras conhecidas;
- cada passo precisa estar em uma ordem que faça sentido;
- alguém precisa interpretar a receita;
- o resultado depende de a receita estar correta e dos ingredientes disponíveis.

No computador, o programa é a receita e os dados são os ingredientes.

## O que uma linguagem define?

Uma linguagem de programação define regras para escrever e organizar o código. Essas regras incluem:

- **sintaxe (*syntax*):** como as instruções devem ser escritas;
- **semântica (*semantics*):** o que cada instrução significa;
- **tipos (*types*):** que tipo de valor pode ser usado, como texto, número ou data;
- **estruturas de controle (*control structures*):** como tomar decisões e repetir ações;
- **funções (*functions*) e métodos (*methods*):** como organizar tarefas reutilizáveis.

Se a sintaxe estiver errada, o programa pode nem ser aceito. Se a sintaxe estiver correta, mas o significado estiver errado, o programa pode executar e mesmo assim produzir um resultado incorreto.

## Exemplo simples

Este exemplo em [[Java]] verifica a idade de uma pessoa:

```java
int idade = 16;

if (idade >= 18) {
    System.out.println("Pode entrar");
} else {
    System.out.println("Ainda não pode entrar");
}
```

O programa:

- guarda `16` na variável `idade`;
- compara a idade com `18`;
- executa um bloco se a condição for verdadeira;
- executa outro bloco se a condição for falsa.

A estrutura `if` é uma decisão. Ela permite que o programa escolha um caminho de acordo com uma condição.

## Como o computador executa o código?

Existem duas formas comuns:

- **compilação (*compilation*):** um compilador transforma o código em outra forma antes da execução. O Java, por exemplo, transforma o código em bytecode, que é executado pela JVM;
- **interpretação (*interpretation*):** um interpretador lê e executa as instruções durante o funcionamento do programa.

Muitas linguagens usam uma combinação dessas ideias. O importante é que o código escrito pela pessoa desenvolvedora precisa ser transformado em operações que o computador consiga realizar.

## Exemplos de linguagens

Cada linguagem costuma ter pontos fortes e ferramentas próprias:

- [[Java]]: muito usado em backend, sistemas corporativos e aplicações de grande porte;
- [[JavaScript]]: muito usado em páginas web e também em aplicações no servidor;
- Python: conhecido pela sintaxe simples e usado em automação, dados e aplicações web;
- SQL: linguagem especializada em consultar e alterar dados em bancos relacionais, como o [[PostgreSQL]].

Não existe uma linguagem melhor para absolutamente todos os problemas. A escolha depende do objetivo, da equipe, do ambiente e das ferramentas disponíveis.

## Linguagem, biblioteca e framework

Uma linguagem fornece as regras básicas para escrever o programa.

Uma [[Biblioteca]] oferece código pronto para realizar tarefas específicas.

Um [[Framework]] fornece uma estrutura maior para organizar a aplicação e controlar parte do seu funcionamento.

Por exemplo, neste projeto:

- [[Java]] é a linguagem;
- [[Quarkus]] é um framework para ajudar a criar o [[Backend]];
- bibliotecas fornecem funcionalidades reutilizáveis;
- [[Maven]] ajuda a organizar o projeto e suas dependências.

## Boas práticas

- escolher uma linguagem adequada ao problema, evitando decisões baseadas apenas em moda;
- aprender a sintaxe e o significado das construções usadas, em vez de apenas copiar código;
- usar nomes claros e organizar o código em pequenas partes;
- seguir o estilo adotado pelo projeto, facilitando a leitura por outras pessoas;
- validar dados recebidos de usuários e outros sistemas;
- escrever testes para verificar comportamentos importantes;
- manter a linguagem e suas dependências em versões suportadas;
- ler mensagens de erro com calma, usando-as para encontrar a causa do problema;
- evitar complexidade desnecessária, preferindo uma solução simples de explicar e manter.

## Resumo

> **Linguagem de programação é uma forma de escrever instruções que transforma ideias humanas em ações executadas pelo computador.**

[[Java]], [[JavaScript]] e Python são linguagens diferentes, cada uma com suas regras, ferramentas e usos comuns.
