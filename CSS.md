# CSS

**CSS** significa *Cascading Style Sheets* (folhas de estilo em cascata). Ele define a aparência, o posicionamento e o comportamento visual dos elementos [[HTML]].

Se o HTML é o esqueleto de uma casa, o CSS é a pintura, a decoração, a iluminação e a organização dos móveis. O HTML diz que existe um botão; o CSS define sua cor, tamanho, espaço e posição.

## Como uma regra CSS funciona

Uma regra CSS possui um **seletor** (*selector*) e declarações dentro de chaves:

```css
button {
  background-color: #2563eb;
  color: white;
  padding: 0.75rem 1rem;
  border: 0;
  border-radius: 0.5rem;
}
```

- `button` é o seletor;
- `background-color`, `color` e os demais nomes são propriedades;
- os valores depois de `:` definem como cada propriedade deve funcionar;
- cada declaração termina com `;`.

Essa regra estiliza todos os elementos `<button>` encontrados na página.

## Seletores comuns

```css
/* Elemento */
p {
  line-height: 1.6;
}

/* Classe reutilizável */
.card {
  padding: 1rem;
}

/* Identificador único */
#menu-principal {
  display: flex;
}

/* Estado do elemento */
button:hover {
  background-color: #1d4ed8;
}
```

Classes são normalmente a melhor escolha para estilos reutilizáveis. IDs podem ser úteis para um elemento específico, mas seu uso excessivo aumenta a dificuldade de manutenção.

## A cascata

“Cascading” significa que o navegador precisa decidir qual regra vence quando várias regras atingem o mesmo elemento. Essa decisão considera, entre outros fatores:

1. se a regra é aplicável;
2. a origem e a importância da regra;
3. a especificidade do seletor;
4. a ordem em que as regras aparecem.

Exemplo:

```css
p {
  color: black;
}

.aviso {
  color: darkred;
}
```

Um parágrafo com `class="aviso"` ficará escuro porque o seletor de classe é mais específico que o seletor de elemento.

Evite resolver conflitos usando `!important` sem necessidade. Ele pode esconder problemas de organização e tornar mudanças futuras mais difíceis.

## Box model

O navegador trata cada elemento como uma caixa (*box*), formada por:

- **content**: conteúdo;
- **padding**: espaço interno;
- **border**: borda;
- **margin**: espaço externo.

```css
.card {
  width: 20rem;
  padding: 1rem;
  border: 1px solid #d1d5db;
  margin: 1rem;
  box-sizing: border-box;
}
```

`box-sizing: border-box` faz a largura incluir o conteúdo, o padding e a borda, deixando o cálculo mais previsível.

## Layout e responsividade

Flexbox ajuda a organizar itens em uma dimensão. Grid ajuda a organizar linhas e colunas.

```css
.produtos {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
}

@media (max-width: 48rem) {
  .produtos {
    grid-template-columns: 1fr;
  }
}
```

Nesse exemplo, três colunas aparecem em telas maiores e uma coluna em telas menores. Esse cuidado é chamado de **responsividade** (*responsive design*).

## Variáveis CSS

Variáveis ajudam a centralizar valores que se repetem:

```css
:root {
  --cor-principal: #2563eb;
  --espaco-padrao: 1rem;
}

.botao-principal {
  background-color: var(--cor-principal);
  padding: var(--espaco-padrao);
}
```

Se a cor ou o espaçamento mudar, basta atualizar a variável. Isso facilita a manutenção de um design consistente.

## CSS no React e no Next.js

O [[React]] e o [[Next.js]] podem usar CSS global, CSS Modules, bibliotecas de componentes ou soluções baseadas em JavaScript.

Com CSS Modules, por exemplo, uma classe pode ficar restrita ao componente:

```jsx
import styles from "./Card.module.css";

export function Card() {
  return <article className={styles.card}>Produto</article>;
}
```

O CSS continua estilizando elementos HTML. A diferença é a forma como os nomes e arquivos são organizados no projeto.

## Boas práticas

- use classes com nomes claros e relacionados à função do componente;
- prefira layout responsivo a tamanhos fixos que quebram em telas pequenas;
- mantenha contraste suficiente entre texto e fundo;
- preserve um indicador visível de foco para quem navega pelo teclado;
- não use apenas cor para transmitir uma informação importante;
- respeite preferências de redução de movimento quando houver animações;
- organize estilos por componentes ou áreas para reduzir conflitos;
- use variáveis para cores, espaçamentos e tamanhos repetidos;
- evite seletores excessivamente específicos e o uso indiscriminado de `!important`;
- teste em navegadores, tamanhos de tela e modos claro e escuro quando aplicável;
- não use CSS como mecanismo de segurança; esconder um botão não impede uma [[Requisição]] direta ao [[Backend]].

CSS cuida da apresentação do [[Frontend]]. Ele trabalha junto com [[HTML]] para formar a interface e com [[React]] e [[Next.js]] para organizar componentes e páginas.
