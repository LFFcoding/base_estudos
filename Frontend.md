# Frontend

**Frontend** é a parte de uma aplicação que o usuário vê e utiliza. Ele também é chamado de camada do cliente (*client-side*), porque normalmente é executado no navegador ou em outro dispositivo do usuário.

O [[HTML]] organiza o conteúdo da página, o [[CSS]] define sua aparência e o [[JavaScript]] adiciona comportamentos. Ferramentas como [[React]] e [[Next.js]] ajudam a construir e organizar essa interface.

Uma analogia é pensar em um restaurante:

- o frontend é o salão, o cardápio e o espaço onde o cliente faz o pedido;
- o [[Backend]] é a cozinha, que recebe o pedido e prepara a resposta;
- o [[Banco de dados]] (*database*) é a despensa onde os ingredientes ficam guardados.

## O que o frontend faz?

O frontend pode:

- mostrar textos, imagens, botões e formulários;
- receber ações do usuário, como cliques e digitação;
- validar informações simples antes de enviá-las;
- enviar [[Requisição|requisições]] (*requests*) para o [[Backend]];
- mostrar respostas, mensagens de erro e indicadores de carregamento;
- adaptar a tela para computadores, tablets e celulares;
- cuidar da aparência e da experiência do usuário (*user experience* ou UX).

O frontend não deve ser responsável por decidir sozinho regras importantes, como se uma senha está correta ou se uma pessoa pode acessar um recurso. Essas decisões precisam ser verificadas pelo [[Backend]], porque o código do frontend pode ser visto e alterado pelo usuário.

## Frontend e backend trabalhando juntos

Quando uma pessoa preenche um formulário, o [[Frontend]] envia os dados para o [[Backend]], normalmente por meio de uma [[API]] (*Application Programming Interface*). O backend processa o pedido e devolve uma resposta (*response*). Depois, o frontend mostra o resultado na tela.

Por exemplo, em um login:

1. o frontend mostra os campos de usuário e senha;
2. a pessoa preenche os campos e clica no botão;
3. o frontend envia os dados para o backend;
4. o backend verifica as informações;
5. o frontend mostra se o acesso foi permitido ou recusado.

## Frontend na stack deste projeto

Nesta [[Stack]], o frontend será construído com:

- [[Next.js]], um [[Framework]] para criar aplicações web;
- [[React]], uma [[Biblioteca]] usada para criar componentes de tela (*UI components*).

Um componente é uma parte reutilizável da interface, como um botão, um campo de formulário ou um menu. Reutilizar componentes ajuda a manter a aplicação organizada e evita repetir código.

## Boas práticas no frontend

- criar telas acessíveis (*accessibility*), para que mais pessoas consigam usar a aplicação;
- adaptar a interface a diferentes tamanhos de tela, usando design responsivo (*responsive design*);
- separar a tela em componentes pequenos, facilitando a reutilização e a manutenção;
- mostrar mensagens claras e estados de carregamento, ajudando o usuário a entender o que está acontecendo;
- validar os dados no frontend para dar uma resposta rápida, mas repetir as validações no backend por segurança;
- otimizar imagens e scripts, melhorando o tempo de carregamento;
- testar os componentes e os principais fluxos, reduzindo erros para o usuário.

## Resumo

> **Frontend é a parte da aplicação que o usuário vê e utiliza para interagir com o sistema.**

Ele é como o salão de um restaurante: apresenta as opções, recebe o pedido e mostra o resultado, enquanto o backend faz o trabalho por trás.
