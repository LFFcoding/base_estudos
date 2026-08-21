# JavaScript

**JavaScript** é uma linguagem de programação. Ela começou sendo muito usada para adicionar comportamento a páginas web, mas hoje também é executada em servidores, ferramentas de terminal, aplicações desktop, dispositivos e serviços.

Uma analogia simples: [[HTML]] é a estrutura de uma casa, [[CSS]] é a aparência, e JavaScript é a parte que faz portas abrirem, luzes responderem e informações mudarem. Fora do navegador, ele também pode funcionar como a linguagem de uma aplicação de [[Backend]].

JavaScript é uma linguagem dinâmica e multiparadigma: permite programação imperativa, orientada a objetos e funcional. A especificação padronizada da linguagem se chama **ECMAScript**. JavaScript não é a mesma linguagem que [[Java]], apesar da semelhança no nome.

## JavaScript, navegador e Node.js

JavaScript é a linguagem; o ambiente que executa a linguagem é chamado de runtime (*ambiente de execução*).

- no navegador, o runtime oferece APIs como `window`, `document`, DOM e recursos de interface;
- no [[Node.js]], o runtime oferece APIs para arquivos, processos, HTTP, streams e terminal;
- em cada ambiente, a linguagem é parecida, mas as APIs disponíveis podem ser diferentes.

Por isso, um código que usa `document.querySelector()` funciona no navegador, mas não necessariamente em um servidor Node.js. Já `node:fs`, usado para ler arquivos no Node.js, não é uma API normal de páginas web.

## Primeiro exemplo

```javascript
const nome = 'Ana';
const mensagem = `Olá, ${nome}!`;

console.log(mensagem);
```

- `const` cria uma variável que não pode ser reatribuída;
- o texto entre crases é um template string e permite inserir valores com `${...}`;
- `console.log` mostra uma mensagem no console.

## Variáveis

As formas modernas de declarar variáveis são `const` e `let`:

```javascript
const limite = 10;
let tentativas = 0;

tentativas += 1;
```

Use `const` quando a variável não precisar receber outro valor e `let` quando a reatribuição fizer parte do comportamento. `var` existe por compatibilidade, mas seu escopo e suas regras antigas podem causar confusão; em código novo, prefira `const` e `let`.

`const` impede trocar o valor guardado na variável, mas não torna automaticamente um objeto imutável:

```javascript
const usuario = { nome: 'Ana' };
usuario.nome = 'Ana Silva'; // permitido: o objeto foi alterado

// usuario = {}; // não permitido: a variável seria reatribuída
```

## Tipos de dados

JavaScript possui valores primitivos como:

- `string`: textos;
- `number`: números comuns;
- `bigint`: números inteiros muito grandes;
- `boolean`: `true` ou `false`;
- `undefined`: valor ainda não definido;
- `null`: ausência intencional de valor;
- `symbol`: identificadores únicos.

Objetos, arrays e funções são valores mais complexos:

```javascript
const nome = 'Ana';
const ativo = true;
const idade = 30;
const endereco = { cidade: 'São Paulo' };
const permissoes = ['ler', 'editar'];
```

JavaScript é dinamicamente tipado: uma variável pode receber valores de tipos diferentes ao longo do tempo. Isso dá flexibilidade, mas torna a validação e os testes importantes.

## Operadores e comparações

Use `===` e `!==` para comparações estritas:

```javascript
const idade = 18;

if (idade >= 18) {
  console.log('maior de idade');
}

console.log(idade === 18); // true
console.log(idade !== 20); // true
```

`===` compara valor e tipo. O operador `==` permite conversões automáticas que podem produzir resultados surpreendentes. Em código novo, prefira comparações estritas, salvo quando houver uma razão documentada.

## Condições e loops

```javascript
const pedidos = [100, 250, 80];

for (const valor of pedidos) {
  if (valor >= 100) {
    console.log('pedido relevante:', valor);
  }
}
```

O `for...of` percorre os valores do array. A condição `if` decide se o bloco será executado.

Para transformar e filtrar arrays, métodos como `map`, `filter` e `reduce` são comuns:

```javascript
const valores = [100, 250, 80];
const grandes = valores.filter((valor) => valor >= 100);
const comTaxa = valores.map((valor) => valor * 1.1);
```

Escolha a forma que fique mais clara. Um `reduce` muito complexo pode ser pior de entender do que um loop explícito.

## Funções

Funções agrupam comportamento reutilizável:

```javascript
function somar(a, b) {
  return a + b;
}

const resultado = somar(2, 3);
```

Também é possível usar uma arrow function (*função de seta*):

```javascript
const multiplicar = (a, b) => a * b;
```

Funções são valores em JavaScript: podem ser guardadas em variáveis, passadas como argumentos e retornadas por outras funções. Isso permite callbacks e estilos de programação funcional.

Evite funções que fazem muitas coisas ao mesmo tempo. Uma função curta, com nome claro e uma responsabilidade bem definida, costuma ser mais fácil de testar.

## Objetos e arrays

Um objeto reúne dados por propriedades:

```javascript
const usuario = {
  id: 42,
  nome: 'Ana',
  ativo: true,
};

console.log(usuario.nome);
console.log(usuario['ativo']);
```

Um array representa uma coleção ordenada:

```javascript
const nomes = ['Ana', 'Bruno', 'Carla'];
console.log(nomes[0]); // Ana
```

Destructuring (*desestruturação*) extrai propriedades ou posições:

```javascript
const { nome, ativo } = usuario;
const [primeiro] = nomes;
```

O spread operator (`...`) cria uma nova estrutura com parte dos valores:

```javascript
const atualizado = { ...usuario, ativo: false };
const maisNomes = [...nomes, 'Diego'];
```

Criar novos objetos em vez de alterar dados compartilhados sem controle pode tornar o fluxo mais previsível, especialmente em interfaces com [[React]].

## `null`, `undefined` e valores ausentes

`undefined` normalmente indica que um valor não foi definido ou não existe naquela propriedade. `null` costuma representar uma ausência intencional:

```javascript
const resposta = {
  telefone: null,
};

console.log(resposta.email); // undefined
```

Defina contratos claros para saber quando um campo pode ser `null`, quando deve ser obrigatório e como a interface deve exibir a ausência.

## Programação assíncrona

Muitas tarefas demoram: chamadas HTTP, leitura de arquivos e consultas ao banco. JavaScript usa callbacks, promises e `async`/`await` para lidar com essas operações sem bloquear o fluxo principal.

Uma **Promise** representa o resultado futuro de uma operação, que pode terminar com sucesso ou falha:

```javascript
async function carregarUsuario() {
  const resposta = await fetch('/api/usuarios/42');

  if (!resposta.ok) {
    throw new Error('Não foi possível carregar o usuário');
  }

  return resposta.json();
}
```

`await` espera o resultado da Promise dentro de uma função `async`. O `try/catch` pode tratar falhas:

```javascript
try {
  const usuario = await carregarUsuario();
  console.log(usuario);
} catch (erro) {
  console.error('Falha:', erro);
}
```

Não ignore rejeições de Promises. Uma falha não tratada pode esconder problemas e deixar uma requisição em estado incorreto.

## Módulos

Módulos dividem o programa em arquivos menores. Com ECMAScript Modules, um arquivo pode exportar uma função:

```javascript
// matematica.js
export function somar(a, b) {
  return a + b;
}
```

Outro arquivo pode importá-la:

```javascript
// app.js
import { somar } from './matematica.js';

console.log(somar(2, 3));
```

Módulos ajudam a evitar variáveis globais e tornam as dependências visíveis. Um projeto deve definir se usará ECMAScript Modules (`import`/`export`) ou CommonJS (`require`/`module.exports`) e manter o padrão consistente.

## JavaScript no frontend

No [[Frontend]], JavaScript pode:

- responder a cliques e formulários;
- alterar elementos da página;
- validar entradas antes do envio;
- fazer uma [[Requisição]] para o backend;
- atualizar a tela com a resposta;
- controlar estado e navegação.

[[React]] é uma biblioteca JavaScript para criar interfaces, e [[Next.js]] oferece uma estrutura para aplicações web que também pode executar código no servidor.

Validação no frontend melhora a experiência, mas não é suficiente para proteger o sistema. O [[Backend]] precisa validar novamente os dados recebidos.

## JavaScript no backend

No backend, JavaScript pode ser executado pelo [[Node.js]]. Frameworks como [[NestJS]] ajudam a organizar módulos, controllers, providers e serviços.

Um fluxo possível é:

```text
Frontend -> Requisição -> Node.js/NestJS -> Prisma -> PostgreSQL
```

JavaScript no servidor ainda precisa de autenticação, autorização, logs, limites, tratamento de erros e testes. A linguagem não fornece essas regras automaticamente.

## JavaScript e JSON

JavaScript e [[JSON]] são relacionados, mas diferentes:

- JavaScript é uma linguagem de programação;
- JSON é um formato de texto para representar e transportar dados;
- um objeto JavaScript pode ser convertido para JSON por serialização;
- JSON recebido pela aplicação precisa ser analisado e validado.

```javascript
const usuario = { id: 42, nome: 'Ana' };
const texto = JSON.stringify(usuario);
const novamente = JSON.parse(texto);
```

JSON não pode executar funções ou comandos. Mesmo assim, dados recebidos de fora devem ser tratados como não confiáveis.

## JavaScript e [[TypeScript]]

[[TypeScript]] é uma linguagem baseada em JavaScript que acrescenta tipos e outras ferramentas. Depois de transformado, o código costuma ser executado como JavaScript por um runtime como [[Node.js]] ou pelo navegador.

[[TypeScript]] pode detectar alguns erros durante o desenvolvimento, mas não substitui validação em tempo de execução. Uma requisição pode conter um texto onde a aplicação esperava um número, mesmo que o código tenha tipos corretos.

## Erros e depuração

Use `Error` para representar falhas e adicione contexto sem expor segredos:

```javascript
function dividir(a, b) {
  if (b === 0) {
    throw new Error('O divisor não pode ser zero');
  }

  return a / b;
}
```

Use `console` com moderação em produção. Prefira logs estruturados, níveis de severidade, identificadores de requisição e ferramentas de observabilidade.

## Testes

JavaScript pode ser testado com o test runner nativo do Node.js ou ferramentas da comunidade. Um teste simples verifica um comportamento:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';

test('soma dois números', () => {
  assert.equal(2 + 3, 5);
});
```

Use [[Testes]] para organizar testes unitários, de integração e de contrato. Teste regras importantes, entradas inválidas, erros e permissões, não apenas o caminho feliz.

## Segurança

- não use `eval` ou execute código recebido de usuários;
- valide e limite entradas, arquivos, URLs e payloads;
- escape ou sanitize conteúdo inserido no HTML para reduzir risco de XSS;
- não coloque tokens, senhas ou chaves no código frontend;
- mantenha dependências atualizadas e revise pacotes antes de instalá-los;
- trate erros sem enviar stack traces ou dados internos para o cliente;
- use cookies, headers e armazenamento do navegador com cuidado;
- no backend, valide autorização em cada operação protegida;
- não confunda código minificado ou ofuscado com código secreto;
- combine essas práticas com as orientações de [[Segurança]].

## Boas práticas

- prefira `const` e use `let` apenas quando precisar reatribuir;
- use comparações estritas (`===` e `!==`);
- mantenha funções pequenas e com nomes claros;
- evite variáveis globais e efeitos colaterais escondidos;
- trate erros síncronos e rejeições assíncronas;
- use módulos para separar responsabilidades;
- valide dados recebidos no limite da aplicação;
- não bloqueie o event loop com trabalho pesado;
- escreva testes para regras de negócio;
- mantenha formatador, linter e versão do runtime padronizados na equipe;
- documente APIs e contratos de dados;
- escolha dependências com manutenção e comunidade confiáveis.

## Resumo

**JavaScript é uma linguagem de programação usada no navegador e em runtimes como Node.js. Ela permite criar interfaces, servidores e ferramentas, mas exige disciplina com tipos, assincronicidade, erros, dependências, segurança e testes.**

## Referências

- [Guia de JavaScript da MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide);
- [Visão geral da linguagem JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Language_overview);
- [Referência de JavaScript da MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference);
- [Especificação ECMAScript](https://ecma-international.org/publications-and-standards/standards/ecma-262/).
