# HTML

**HTML** significa *HyperText Markup Language* (linguagem de marcação de hipertexto). Ele define a estrutura e o significado do conteúdo de uma página web.

HTML é como o esqueleto e a planta de uma casa: informa quais partes existem, como elas se organizam e qual é a função de cada uma. O [[CSS]] cuida da aparência, enquanto o [[JavaScript]] cuida dos comportamentos e interações.

## HTML não é uma linguagem de programação

HTML é uma **linguagem de marcação** (*markup language*). Ele usa elementos e tags para descrever o conteúdo, mas não possui a mesma lógica de decisão e repetição de uma linguagem de programação.

Exemplo:

```html
<p>Olá, mundo!</p>
```

Nesse exemplo, `<p>` indica o início de um parágrafo e `</p>` indica o fim. O navegador interpreta a marcação e mostra o texto na página.

## Estrutura de um documento

Uma página HTML básica pode ser escrita assim:

```html
<!doctype html>
<html lang="pt-BR">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cadastro</title>
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <header>
      <h1>Cadastro de produto</h1>
    </header>

    <main>
      <p>Preencha os dados do produto.</p>
    </main>
  </body>
</html>
```

- `<!doctype html>` informa que o documento usa HTML moderno;
- `<html>` envolve a página inteira;
- `<head>` guarda metadados e referências a recursos;
- `<body>` contém o que pode ser apresentado à pessoa;
- `<header>` representa o cabeçalho;
- `<main>` representa o conteúdo principal;
- `<h1>` é o título principal;
- `<p>` é um parágrafo;
- `<link>` conecta o HTML a uma folha de estilos CSS.

## Elementos, tags e atributos

Um **elemento** é formado pela tag de abertura, pelo conteúdo e, quando necessário, pela tag de fechamento:

```html
<a href="/produtos">Ver produtos</a>
```

`<a>` é a tag do link. `href` é um **atributo** que informa o destino. O texto entre as tags é o conteúdo que a pessoa vê.

Alguns elementos não envolvem conteúdo e podem ser escritos sozinhos:

```html
<img src="produto.png" alt="Tênis azul" />
```

O atributo `alt` descreve a imagem para pessoas que usam leitores de tela ou quando a imagem não pode ser carregada.

## HTML semântico

HTML semântico usa elementos que explicam a finalidade do conteúdo:

- `<header>`: cabeçalho;
- `<nav>`: navegação;
- `<main>`: conteúdo principal;
- `<section>`: seção relacionada;
- `<article>`: conteúdo independente;
- `<aside>`: conteúdo complementar;
- `<footer>`: rodapé;
- `<button>`: ação que pode ser executada;
- `<form>`: formulário para coletar dados.

Usar `<button>` para uma ação é melhor do que usar uma `<div>` com aparência de botão. O navegador, os leitores de tela e o teclado conseguem entender melhor o elemento correto.

## Formulários

Um formulário pode coletar dados e enviá-los ao servidor:

```html
<form action="/produtos" method="post">
  <label for="nome">Nome</label>
  <input id="nome" name="nome" type="text" required />
  <button type="submit">Salvar</button>
</form>
```

- `label` explica o campo e está associado a ele por `for` e `id`;
- `name` identifica o valor enviado;
- `required` indica que o preenchimento é necessário no navegador;
- `method="post"` indica uma operação que envia dados.

A validação do navegador ajuda a pessoa, mas o [[Backend]] precisa validar tudo novamente. O [[Frontend]] não deve ser a única proteção.

## HTML, frontend e componentes

O HTML é a base visual do [[Frontend]]. O [[React]] pode criar elementos HTML usando JSX, e o [[Next.js]] pode organizar páginas, rotas e renderização.

JSX parece HTML, mas é transformado pelo React. Por isso, pequenas diferenças existem, como usar `className` em vez de `class` em componentes React.

Uma página também pode carregar dados do backend por uma [[Requisição]] e depois atualizar partes do HTML com [[JavaScript]].

## Boas práticas

- use HTML semântico, escolhendo o elemento pela função e não apenas pela aparência;
- mantenha uma ordem lógica de títulos, começando pelo `<h1>`;
- associe todo campo de formulário a um `<label>`;
- forneça `alt` descritivo para imagens importantes;
- garanta navegação por teclado e foco visível;
- informe o idioma com `lang`, como `lang="pt-BR"`;
- use `button` para ações e `a` para navegação;
- valide o HTML e teste em tamanhos de tela diferentes;
- escape ou sanitize conteúdo dinâmico para reduzir riscos de XSS;
- não coloque senhas, tokens ou regras de autorização no HTML enviado ao navegador.

HTML define a estrutura e o significado da página. Uma interface completa normalmente combina HTML, [[CSS]] e [[JavaScript]], com ferramentas como [[React]] e [[Next.js]].
