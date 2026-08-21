# TypeScript

**TypeScript** é uma linguagem baseada em [[JavaScript]] que adiciona sintaxe para descrever tipos. Ela ajuda o editor e o compilador a encontrar possíveis erros antes de o programa ser executado.

Uma analogia: JavaScript é como escrever uma receita com liberdade para trocar ingredientes; TypeScript permite anotar antecipadamente quais ingredientes cada etapa espera. Essas anotações ajudam a encontrar erros, mas ainda é necessário conferir os ingredientes reais quando a receita chega.

O código TypeScript normalmente é transformado em JavaScript. O navegador e runtimes como [[Node.js]] executam o JavaScript gerado. TypeScript não é banco de dados, framework ou runtime.

## TypeScript e JavaScript

TypeScript foi criado para trabalhar junto com JavaScript:

- a sintaxe básica é parecida;
- arquivos `.ts` e `.tsx` usam recursos de JavaScript;
- o compilador verifica tipos e gera JavaScript;
- bibliotecas JavaScript podem ser usadas em projetos TypeScript;
- aprender [[JavaScript]] continua sendo importante.

Sem tipos, este problema pode aparecer somente durante a execução:

```javascript
function saudacao(usuario) {
  return 'Olá, ' + usuario.nome;
}

saudacao('Bruno');
```

Com TypeScript, descrevemos o formato esperado:

```typescript
type Usuario = {
  nome: string;
};

function saudacao(usuario: Usuario): string {
  return 'Olá, ' + usuario.nome;
}

// saudacao('Bruno'); // erro antes da execução
```

O compilador avisa que uma string não possui o formato `Usuario`. Isso reduz erros entre partes do código, mas não valida automaticamente dados vindos da internet.

## Tipos básicos

```typescript
const nome: string = 'Ana';
const idade: number = 30;
const ativo: boolean = true;
const ids: number[] = [1, 2, 3];
```

Na maioria dos casos, TypeScript consegue inferir o tipo:

```typescript
const cidade = 'São Paulo'; // string
const quantidade = 3; // number
```

Não escreva tipos em todas as linhas sem necessidade. Use anotações quando elas melhorarem a comunicação ou descreverem uma entrada, saída ou contrato.

## Objetos, type e interface

Um `type` pode descrever as propriedades de um objeto:

```typescript
type Produto = {
  id: number;
  nome: string;
  preco: number;
  descricao?: string;
};
```

O `?` indica uma propriedade opcional. Uma `interface` também descreve um formato:

```typescript
interface Pedido {
  id: number;
  total: number;
  pago: boolean;
}
```

`type` e `interface` resolvem muitos problemas parecidos. A equipe deve escolher um padrão consistente.

## Union types e narrowing

Uma união (*union type*) diz que um valor pode ter mais de um tipo:

```typescript
function mostrarId(id: number | string): string {
  if (typeof id === 'number') {
    return `Número: ${id}`;
  }

  return `Texto: ${id}`;
}
```

O `typeof` permite fazer *narrowing* (refinamento). Depois da condição, TypeScript entende qual tipo existe em cada caminho.

Use esse refinamento para tratar casos diferentes explicitamente, em vez de presumir que um valor sempre possui determinada propriedade.

## Funções e generics

TypeScript permite declarar tipos dos parâmetros e do retorno:

```typescript
function somar(a: number, b: number): number {
  return a + b;
}
```

Generics (*genéricos*) permitem reutilizar uma função sem perder a relação entre os tipos:

```typescript
function primeiro<T>(valores: T[]): T | undefined {
  return valores[0];
}

const numero = primeiro([1, 2, 3]); // number | undefined
const texto = primeiro(['a', 'b']); // string | undefined
```

Use generics quando eles tornarem o contrato mais preciso. Um generic excessivamente complexo pode dificultar a leitura.

## `any`, `unknown` e `strict`

- **`any`:** desliga boa parte da verificação de tipos. Use somente com uma razão clara.
- **`unknown`:** representa um valor desconhecido que precisa ser verificado antes de ser usado.
- **`strict`:** ativa verificações mais rigorosas no compilador.

Exemplo com `unknown`:

```typescript
function registrarErro(erro: unknown): void {
  if (erro instanceof Error) {
    console.error(erro.message);
    return;
  }

  console.error('Erro desconhecido');
}
```

Evite substituir todo problema com `any`. Isso remove justamente a proteção que motivou a escolha do TypeScript.

## Compilação e `tsconfig.json`

O arquivo `tsconfig.json` configura o compilador:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noImplicitAny": true,
    "outDir": "dist",
    "sourceMap": true
  },
  "include": ["src/**/*.ts"]
}
```

- `target` define uma versão de JavaScript de saída;
- `module` e `moduleResolution` definem o tratamento dos módulos;
- `strict` e `noImplicitAny` aumentam a verificação;
- `outDir` define a pasta dos arquivos gerados;
- `sourceMap` ajuda a depurar o código original.

O compilador pode ser instalado e executado assim:

```bash
npm install --save-dev typescript
npx tsc --init
npx tsc
node dist/index.js
```

`tsc` verifica os tipos e gera JavaScript conforme o `tsconfig.json`. O comando `node` executa o arquivo gerado.

## TypeScript em APIs

TypeScript pode descrever o formato de uma resposta de [[Requisição|requisição]]:

```typescript
type UsuarioResponse = {
  id: number;
  nome: string;
};

async function buscarUsuario(): Promise<UsuarioResponse> {
  const resposta = await fetch('/api/usuarios/42');

  if (!resposta.ok) {
    throw new Error('Falha ao buscar usuário');
  }

  return resposta.json() as Promise<UsuarioResponse>;
}
```

O tipo documenta o formato esperado, mas `as` não verifica o conteúdo real recebido pela rede. Para isso, use validação em tempo de execução.

## TypeScript no frontend e backend

Em [[React]] e [[Next.js]], arquivos `.tsx` permitem usar JSX com tipos:

```tsx
type SaudacaoProps = {
  nome: string;
};

export function Saudacao({ nome }: SaudacaoProps) {
  return <h1>Olá, {nome}!</h1>;
}
```

No [[Backend]], o [[NestJS]] usa TypeScript com frequência, e o [[Prisma]] gera tipos para consultas e modelos:

```text
[[Frontend]] -> [[Requisição]] -> [[NestJS]]/TypeScript -> [[Prisma]] -> [[PostgreSQL]]
```

Uma aplicação ainda precisa validar o corpo da requisição, autenticar o usuário, autorizar a operação e tratar falhas de banco. Tipos não substituem essas etapas.

## TypeScript e Java

TypeScript e [[Java]] possuem tipagem, mas funcionam de formas diferentes:

| Característica | TypeScript | Java |
| --- | --- | --- |
| Relação | baseado em [[JavaScript]] | linguagem independente |
| Execução | normalmente vira JavaScript | bytecode executado na JVM |
| Ecossistema | npm, Node.js, React e NestJS | [[Maven]], JVM e [[Quarkus]] |

A stack principal deste projeto usa Java 17. TypeScript aparece como alternativa no ecossistema web.

## Testes

Tipos detectam alguns problemas, mas testes verificam comportamento:

```typescript
function calcularTotal(valor: number, taxa: number): number {
  return valor + taxa;
}

const total = calcularTotal(100, 10);
// Um teste deve verificar se total é 110.
```

Use [[Testes]] para cobrir regras de negócio, respostas de API, validação, permissões e integrações. Um programa pode estar perfeitamente tipado e ainda calcular o resultado errado.

## Segurança e boas práticas

- não trate tipos como validação de entrada externa;
- valide dados de [[Requisição|requisições]], filas e arquivos em tempo de execução;
- evite `any` em áreas importantes do sistema;
- não use `as` para esconder erros que deveriam ser investigados;
- não coloque segredos em arquivos `.ts` enviados ao frontend;
- não confie em tipos enviados pelo cliente;
- revise dependências e arquivos de declaração de terceiros;
- execute verificação de tipos e testes no CI;
- combine TypeScript com as práticas de [[Segurança]].

## Resumo

**TypeScript é uma linguagem baseada em JavaScript que adiciona tipos e ferramentas de verificação. Ele ajuda a encontrar erros antes da execução e melhora a organização de aplicações grandes, mas o código final ainda precisa virar JavaScript e os dados externos continuam precisando de validação em tempo de execução.**

## Referências oficiais

- [Documentação oficial do TypeScript](https://www.typescriptlang.org/docs/);
- [Handbook do TypeScript](https://www.typescriptlang.org/docs/handbook/intro.html);
- [Tipos do dia a dia](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html);
- [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html);
- [Boas práticas e cuidados](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html).
