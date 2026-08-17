# Next.js

**Next.js** é um [[Framework]] para criar aplicações web usando [[React]]. Ele fornece uma estrutura pronta para organizar páginas, rotas, carregamento de dados e publicação do [[Frontend]].

Nesta [[Stack]], o Next.js será usado com React para construir a parte visual da aplicação. Ele pode conversar com o [[Backend]] por meio de uma [[API]].

Uma analogia é construir uma casa usando peças de LEGO:

- React fornece as peças da interface;
- Next.js fornece a planta, os caminhos entre os cômodos e regras de construção;
- o frontend é a casa que o usuário visita;
- o backend fornece os serviços e dados usados pelos cômodos.

## O que o Next.js facilita?

O Next.js ajuda a:

- criar páginas e rotas;
- organizar arquivos por convenções;
- renderizar páginas no servidor (*server-side rendering*);
- gerar páginas estáticas (*static generation*);
- executar partes da interface no navegador (*client-side rendering*);
- criar layouts compartilhados;
- carregar dados;
- otimizar imagens e fontes;
- definir metadados para mecanismos de busca;
- preparar a aplicação para produção.

Ele permite escolher onde cada parte deve ser executada. Algumas partes podem ser processadas no servidor, e outras precisam de interação no navegador.

## Criando uma página

Em um projeto que usa o App Router, um arquivo `app/page.tsx` pode representar a página inicial:

```tsx
export default function HomePage() {
    return (
        <main>
            <h1>Minha aplicação</h1>
            <p>Bem-vindo!</p>
        </main>
    );
}
```

O Next.js usa a localização e o nome dos arquivos para organizar rotas. O componente exportado como padrão representa o conteúdo daquela página.

## Componentes do navegador

Quando um componente precisa reagir a cliques, digitação ou outros eventos do navegador, ele pode ser marcado com `"use client"`:

```tsx
"use client";

import { useState } from "react";

export default function Contador() {
    const [valor, setValor] = useState(0);

    return (
        <button onClick={() => setValor(valor + 1)}>
            Valor: {valor}
        </button>
    );
}
```

Essa diretiva informa que o componente precisa ser executado no navegador porque usa estado e evento de clique.

Quando não existe necessidade de interação no navegador, manter o componente no servidor pode reduzir o JavaScript enviado ao cliente.

## Buscando dados

Uma página pode buscar dados de uma API do backend:

```tsx
type Usuario = {
    id: number;
    nome: string;
};

async function buscarUsuarios(): Promise<Usuario[]> {
    const resposta = await fetch("http://localhost:8080/usuarios");

    if (!resposta.ok) {
        throw new Error("Não foi possível buscar os usuários");
    }

    return resposta.json();
}
```

O código verifica `resposta.ok` antes de usar os dados. Em produção, o endereço do backend deve vir de uma configuração de ambiente, e não ficar fixo no código.

## Comandos principais

Dentro de um projeto Next.js, comandos comuns são:

```bash
npm install
npm run dev
npm run lint
npm run build
npm start
```

- `npm install` instala as dependências;
- `npm run dev` inicia o modo de desenvolvimento;
- `npm run lint` verifica problemas de estilo e qualidade;
- `npm run build` cria a versão de produção;
- `npm start` inicia a versão construída.

O arquivo `package.json` registra scripts e dependências do projeto.

## Next.js e React

React é uma [[Biblioteca]] para criar componentes. Next.js é um framework que usa React e adiciona uma estrutura para a aplicação web.

Uma comparação simples:

- React fornece as peças para montar a interface;
- Next.js define como organizar e executar a casa inteira.

## Next.js e backend

O Next.js pode renderizar a interface e também oferecer alguns recursos do lado do servidor, mas, nesta arquitetura, o [[Backend]] principal será construído com [[Java]], [[Quarkus]] e [[Maven]].

O frontend deve chamar a API do backend e não acessar diretamente o [[PostgreSQL]]. Essa separação facilita segurança, manutenção e organização das responsabilidades.

## Boas práticas

- preferir componentes de servidor quando não houver necessidade de interação no navegador;
- usar `"use client"` somente nos componentes que realmente precisam dele;
- tratar estados de carregamento, erro e ausência de dados;
- validar as respostas recebidas da API;
- colocar URLs e configurações por ambiente em variáveis adequadas;
- nunca expor segredos em variáveis públicas do navegador, como as que usam `NEXT_PUBLIC_`;
- usar otimização de imagens e metadados quando fizer sentido;
- manter páginas e componentes pequenos e fáceis de testar;
- executar lint, testes e build antes da publicação;
- não confiar em validações feitas apenas no frontend; o backend também deve validar;
- usar acessibilidade e HTML semântico, facilitando o uso por diferentes pessoas;
- manter dependências atualizadas com cuidado e revisar mudanças importantes.

## Resumo

> **Next.js é um framework que usa React para criar aplicações web organizadas, renderizadas de diferentes formas e preparadas para produção.**

React cuida dos componentes da interface; Next.js fornece a estrutura maior da aplicação.
