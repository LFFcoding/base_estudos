# React

**React** é uma [[Biblioteca]] [[JavaScript]] usada para criar interfaces de usuário (*user interfaces* ou UI).

Ele permite dividir uma tela em componentes reutilizáveis, como botões, menus, formulários e cartões. Nesta [[Stack]], o React será usado no [[Frontend]], junto com o [[Next.js]].

Uma analogia é montar uma tela com peças de LEGO:

- cada componente é uma peça;
- props são informações passadas para a peça;
- state é o estado interno da peça;
- a tela inteira é a combinação das peças;
- quando um estado muda, o React atualiza a parte necessária da interface.

## Componentes

Um componente é uma função que retorna uma descrição da tela. Exemplo:

```tsx
type SaudacaoProps = {
    nome: string;
};

export function Saudacao({ nome }: SaudacaoProps) {
    return <h1>Olá, {nome}!</h1>;
}
```

Esse componente recebe a propriedade `nome` (*prop*) e mostra uma saudação. As props são informações fornecidas pelo componente pai.

## Estado

O estado (*state*) guarda uma informação que pode mudar durante o uso da tela:

```tsx
import { useState } from "react";

export function Contador() {
    const [contador, setContador] = useState(0);

    return (
        <button onClick={() => setContador(contador + 1)}>
            Cliques: {contador}
        </button>
    );
}
```

Nesse exemplo:

- `useState(0)` cria um estado iniciado em `0`;
- `contador` guarda o valor atual;
- `setContador` altera o valor;
- `onClick` reage ao clique;
- depois da alteração, o React renderiza a interface novamente.

## JSX

React costuma usar JSX, uma sintaxe que permite escrever algo parecido com [[HTML]] dentro do código [[JavaScript]] ou [[TypeScript]]. A aparência desses elementos pode ser definida com [[CSS]].

```tsx
const mensagem = <p>Estudando React</p>;
```

JSX não é [[HTML]] puro. Ele é transformado em chamadas que o React usa para criar a interface.

## React e backend

O React cuida principalmente da interface. Para buscar ou enviar dados, ele pode fazer uma [[Requisição]] para uma [[API]] do [[Backend]]:

```tsx
async function buscarUsuarios() {
    const resposta = await fetch("/api/usuarios");
    return resposta.json();
}
```

O exemplo chama uma API e transforma a resposta em dados [[JavaScript]]. Em uma aplicação real, também é necessário tratar carregamento, erros, autenticação e validação.

## React é uma biblioteca, não um framework

O React fornece componentes e mecanismos para criar interfaces, mas não define sozinho toda a estrutura da aplicação.

O [[Next.js]] é um [[Framework]] construído sobre React. Ele acrescenta recursos como rotas, renderização no servidor, otimização e convenções de projeto.

## Boas práticas

- criar componentes pequenos e com responsabilidades claras;
- usar nomes que expliquem o papel de cada componente;
- manter o estado no menor componente que realmente precisa dele;
- não alterar o estado diretamente; usar a função de atualização;
- fornecer uma `key` estável ao renderizar listas;
- criar elementos HTML acessíveis, com labels, textos alternativos e navegação por teclado;
- tratar estados de carregamento, sucesso, vazio e erro;
- evitar efeitos (`useEffect`) quando uma informação pode ser calculada diretamente;
- não colocar senhas, tokens privados ou regras de autorização no código enviado ao navegador;
- testar componentes e fluxos importantes;
- evitar componentes gigantes, dividindo a interface quando a leitura ficar difícil.

## Resumo

> **React é uma biblioteca [[JavaScript]] que ajuda a criar interfaces usando componentes reutilizáveis e estados controlados.**

Ele cuida da interface; outras ferramentas, como Next.js, complementam a estrutura da aplicação web.
