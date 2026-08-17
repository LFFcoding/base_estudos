# Biblioteca

Em programação, uma **biblioteca** (*library*) é um conjunto de códigos prontos e reutilizáveis (*reusable code*) que podemos usar dentro de uma aplicação.

Ela funciona como uma caixa de ferramentas. Em vez de construir uma ferramenta do zero, o programador escolhe uma função da biblioteca e a chama quando precisa.

Por exemplo, se uma aplicação precisa transformar uma data em texto, podemos usar uma biblioteca que já sabe fazer isso. Assim, o programador se concentra nas regras do sistema, em vez de reescrever uma solução conhecida.

## O que uma biblioteca pode oferecer?

Uma biblioteca pode ajudar a:

- formatar datas, números e textos;
- converter dados para [[JSON]] (*JavaScript Object Notation*) e de JSON para objetos;
- fazer [[Requisição|requisições]] HTTP (*HTTP requests*) para outros sistemas;
- validar informações (*validation*);
- trabalhar com arquivos;
- criar telas e componentes (*components*);
- facilitar o acesso ao banco de dados.

Cada biblioteca costuma resolver um tipo de problema. Um projeto pode usar várias bibliotecas diferentes, escolhendo uma ferramenta para cada necessidade.

## Como uma biblioteca é usada?

O programador adiciona a biblioteca ao projeto e chama seus métodos ou funções no momento necessário. No [[Backend]], por exemplo, uma biblioteca pode ser usada para transformar um objeto Java em JSON antes de enviar uma resposta ao [[Frontend]].

Ferramentas como [[Maven]] ajudam a declarar e baixar as bibliotecas que o [[Backend]] precisa. Essas bibliotecas são chamadas de dependências (*dependencies*) do projeto.

## Biblioteca e framework

Uma biblioteca é como uma ferramenta que o programador pega e usa quando precisa. O fluxo principal continua sendo controlado pelo código da aplicação.

Um [[Framework]] é uma estrutura maior. Ele organiza o projeto e pode controlar parte do fluxo, chamando o código do programador nos momentos definidos por suas regras.

Uma forma curta de lembrar:

> Com uma biblioteca, seu código chama a ferramenta. Com um framework, a estrutura do framework orienta o funcionamento do seu código.

## Boas práticas ao usar bibliotecas

Antes de adicionar uma biblioteca, é importante:

- escolher uma biblioteca que resolva realmente o problema, evitando código e dependências desnecessárias;
- verificar se ela continua recebendo atualizações, reduzindo riscos de segurança;
- conferir se é compatível com o projeto, evitando conflitos entre versões;
- ler sua documentação (*documentation*), aprendendo a usá-la corretamente;
- verificar sua licença (*license*), garantindo que seu uso é permitido;
- revisar as dependências que ela traz, pois uma biblioteca pode adicionar outros códigos ao projeto;
- atualizar versões com cuidado e testar a aplicação depois, evitando que uma mudança quebre funcionalidades.

## Resumo

> **Biblioteca é um conjunto de códigos reutilizáveis que o programador chama para realizar tarefas específicas.**
