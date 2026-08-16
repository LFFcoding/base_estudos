# Framework

Um **framework** é uma estrutura pronta de desenvolvimento (*development framework*) que ajuda a criar aplicações. Ele oferece ferramentas, convenções (*conventions*), padrões (*patterns*) e um jeito organizado de trabalhar, para que o programador não precise começar tudo do zero.

Uma analogia é construir uma casa usando uma planta e alguns materiais já preparados. Você ainda decide os cômodos, as cores e os detalhes, mas já recebe uma estrutura que facilita o trabalho.

## O que um framework oferece?

Um framework pode ajudar com:

- organização das pastas e do código;
- recebimento e envio de [[Requisição|requisições]] (*requests*);
- criação de páginas ou rotas (*routes*);
- conexão com o [[Banco de dados]];
- validação de informações (*validation*);
- segurança e controle de acesso (*access control*);
- configurações para executar e publicar a aplicação.

Além de fornecer ferramentas, o framework também define alguns caminhos e padrões. Em muitos casos, ele controla parte do fluxo da aplicação (*control flow*) e chama o código do programador no momento certo. Isso ajuda várias pessoas a trabalharem no mesmo projeto sem que cada uma organize o código de um jeito completamente diferente.

## Framework na stack deste projeto

Nesta [[Stack]], alguns exemplos são:

- [[Quarkus]], usado para construir o [[Backend]] em [[Java 17]];
- [[Next.js]], usado para construir o [[Frontend]] com [[React]].

## Framework e [[Biblioteca]] são a mesma coisa?

Não exatamente.

Uma **[[Biblioteca]]** é como uma ferramenta que você chama quando precisa. Por exemplo, você pode usar uma biblioteca para formatar datas ou validar um texto.

Um **framework** é mais parecido com uma estrutura que organiza o trabalho inteiro. Ele define várias regras do projeto e, em muitos casos, chama o código do programador no momento certo.

Uma forma curta de lembrar:

> Com uma biblioteca, seu código chama a ferramenta. Com um [[Framework]], a estrutura do framework orienta o funcionamento do seu código.

## Por que usar um framework?

Frameworks ajudam a:

- desenvolver mais rápido;
- evitar repetir soluções comuns;
- manter o projeto organizado;
- seguir boas práticas;
- facilitar a manutenção do sistema.

Eles não resolvem todos os problemas automaticamente. Ainda é necessário entender as regras do sistema e escrever o código específico da aplicação.

## Boas práticas ao usar um framework

- seguir as convenções do framework, deixando o projeto familiar para outras pessoas;
- ler a documentação antes de usar uma funcionalidade, evitando soluções incorretas ou desatualizadas;
- manter as regras de negócio separadas da estrutura do framework, facilitando testes e futuras mudanças;
- atualizar as dependências (*dependencies*) com cuidado, verificando compatibilidade e possíveis mudanças;
- usar apenas os recursos necessários, mantendo a aplicação mais simples e fácil de manter;
- conhecer o que acontece por trás das facilidades do framework, evitando depender de “mágica” que não conseguimos explicar.

## Resumo

> **Framework é uma estrutura de ferramentas e padrões que facilita a construção de aplicações.**
