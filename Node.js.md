# Node.js

**Node.js** é um ambiente de execução (*runtime*) que permite executar JavaScript fora do navegador. Ele é usado para criar APIs, servidores, ferramentas de terminal, scripts e aplicações que precisam conversar com a rede ou com o sistema operacional.

Uma analogia simples: JavaScript é a linguagem; o Node.js é o motor que permite executar essa linguagem em um servidor ou no terminal. O navegador também possui um motor para JavaScript, mas oferece APIs diferentes, como `document` e `window`. O Node.js oferece APIs para arquivos, rede, processos, streams e sistema operacional.

A documentação oficial descreve Node.js como um runtime JavaScript construído sobre o motor V8. Ele é multiplataforma, de código aberto e possui APIs para HTTP, arquivos, módulos, criptografia, testes e outras tarefas de servidor.

## Node.js não é JavaScript

Eles estão relacionados, mas não são a mesma coisa:

- **JavaScript:** linguagem de programação;
- **V8:** motor que interpreta e executa JavaScript;
- **Node.js:** runtime que usa V8 e acrescenta APIs para executar JavaScript fora do navegador;
- **NestJS:** framework que organiza uma aplicação executada no [[Node.js]];
- **[[Prisma]]:** toolkit usado por aplicações Node.js e TypeScript para acessar bancos.

Também não é um banco de dados, servidor físico ou hospedagem. Uma aplicação Node.js ainda precisa ser executada em uma máquina, contêiner ou serviço de nuvem.

## Para que serve?

Node.js pode ser usado para:

- criar APIs HTTP e aplicações de [[Backend]];
- executar servidores em tempo real com WebSockets;
- consumir e publicar mensagens em brokers como [[RabbitMQ]];
- criar ferramentas de linha de comando;
- automatizar tarefas e processar arquivos;
- executar ferramentas de build usadas por aplicações de [[Frontend]];
- criar funções e serviços pequenos;
- trabalhar com streams de dados, uploads e downloads.

Um fluxo comum é:

```text
cliente -> servidor Node.js -> regra da aplicação -> banco ou serviço externo
```

O Node.js não define a arquitetura inteira. Um framework como [[NestJS]] pode organizar controllers, módulos e services; a aplicação ainda precisa escolher banco, autenticação, validação, logs e testes.

## Event loop e operações assíncronas

Node.js possui uma arquitetura assíncrona e orientada a eventos (*event-driven*). Em vez de ficar parado esperando uma operação de entrada e saída (*I/O*) terminar, o programa pode registrar o que deve acontecer depois e continuar atendendo outras tarefas.

Uma analogia é um atendente de restaurante:

- ele recebe o pedido;
- entrega o pedido à cozinha;
- enquanto a cozinha trabalha, atende outra mesa;
- quando a comida fica pronta, ele volta ao pedido correspondente.

O **event loop** coordena esses callbacks, promises e eventos. Isso funciona muito bem quando a aplicação passa bastante tempo esperando rede, arquivos ou banco.

```javascript
console.log('início');

setTimeout(() => {
  console.log('terminou depois');
}, 1000);

console.log('fim');
```

O resultado começa com `início` e `fim`; a mensagem do `setTimeout` aparece depois. O timer não bloqueia a execução do restante do script.

### O que significa não bloquear?

Uma operação bloqueante mantém o fluxo ocupado até terminar. Uma operação não bloqueante inicia o trabalho e permite que o event loop cuide de outras tarefas enquanto aguarda o resultado.

Isso não significa que Node.js seja magicamente mais rápido em qualquer situação. Código que executa cálculos muito pesados na thread principal pode bloquear o event loop e atrasar todas as requisições. Para tarefas de CPU, considere workers, processos separados, filas como [[RabbitMQ]] ou uma arquitetura adequada.

Node.js também possui recursos para usar outras threads e processos, mas o benefício principal do modelo tradicional está no bom atendimento de muitas operações de I/O concorrentes.

## Criando um servidor HTTP

Node.js possui um módulo HTTP nativo. Um servidor mínimo pode ser escrito assim:

```javascript
import { createServer } from 'node:http';

const server = createServer((request, response) => {
  response.writeHead(200, { 'Content-Type': 'application/json' });
  response.end(JSON.stringify({ mensagem: 'Olá, Node.js!' }));
});

server.listen(3000, '127.0.0.1', () => {
  console.log('Servidor em http://127.0.0.1:3000');
});
```

Esse código cria um servidor local na porta `3000` e devolve uma resposta em [[JSON]]. Em um projeto real, um framework como [[NestJS]] facilita rotas, validação, erros, autenticação e organização dos módulos.

Para testar o servidor, use [[cURL]]:

```bash
curl http://127.0.0.1:3000
```

## Módulos

Módulos dividem o código em partes reutilizáveis. Node.js possui módulos nativos, como:

- `node:http` para HTTP;
- `node:fs` e `node:fs/promises` para arquivos;
- `node:path` para caminhos de arquivos;
- `node:crypto` para operações criptográficas;
- `node:events` para eventos;
- `node:test` para testes.

Também é possível criar módulos próprios e instalar pacotes da comunidade.

Node.js suporta dois estilos principais de módulos:

### ECMAScript Modules

Usam `import` e `export`:

```javascript
import { readFile } from 'node:fs/promises';

const conteudo = await readFile('notas.txt', 'utf8');
console.log(conteudo);
```

### CommonJS

Usa `require` e `module.exports`:

```javascript
const { readFile } = require('node:fs/promises');

readFile('notas.txt', 'utf8').then(console.log);
```

Os dois estilos existem, mas um projeto deve escolher e documentar seu padrão. Misturar módulos sem entender a configuração pode gerar erros de importação.

## npm e package.json

O **npm** é o gerenciador de pacotes mais associado ao Node.js. Ele instala bibliotecas, executa scripts e registra as dependências de um projeto.

Um projeto pode começar assim:

```bash
mkdir api-estudos
cd api-estudos
npm init -y
npm install lodash
npm install --save-dev typescript
```

- `npm init -y` cria um `package.json` padrão;
- `npm install lodash` adiciona uma dependência usada em execução;
- `npm install --save-dev typescript` adiciona uma ferramenta usada no desenvolvimento.

Exemplo de scripts no `package.json`:

```json
{
  "scripts": {
    "dev": "node --watch src/server.js",
    "start": "node dist/server.js",
    "test": "node --test"
  }
}
```

Depois, os scripts podem ser executados com:

```bash
npm run dev
npm test
```

O arquivo `package-lock.json` registra versões concretas das dependências. Em automações e CI, `npm ci` instala a partir desse lockfile de forma previsível.

Não versionar `node_modules`: a pasta contém dependências instaladas e pode ser recriada pelo npm a partir do `package.json` e do lockfile.

## JavaScript e TypeScript

Node.js executa JavaScript. Muitos projetos usam TypeScript para adicionar tipos e detectar parte dos erros antes da execução.

TypeScript normalmente precisa ser transformado ou executado por uma ferramenta compatível antes de chegar ao runtime. Frameworks como [[NestJS]] costumam organizar esse processo.

Tipos ajudam, mas não validam automaticamente os dados recebidos pela rede. Uma requisição ainda precisa de validação em tempo de execução, autenticação e autorização.

## Variáveis de ambiente

Configurações que mudam entre ambientes podem ser lidas de variáveis de ambiente:

```javascript
const porta = Number(process.env.PORT ?? 3000);
const ambiente = process.env.NODE_ENV ?? 'development';

console.log({ porta, ambiente });
```

Use variáveis para portas, URLs de banco, chaves e configurações. Nunca coloque segredos diretamente no código ou no Git. Proteja esses valores com as práticas de [[Segurança]].

## Node.js com backend e banco

Node.js fornece o ambiente de execução, mas o sistema completo possui outras camadas: **[[Frontend]] → [[Requisição]] → aplicação Node.js → [[Prisma]] → [[PostgreSQL]]**.

Esse é apenas um exemplo. A aplicação poderia usar [[NestJS]], outro framework, SQL direto ou um banco como [[MongoDB]].

O Node.js não substitui o banco de dados. Ele executa o código que aplica regras, valida dados, consulta o banco e devolve respostas.

## Node.js e a stack principal

A stack principal deste repositório usa [[Java|Java 17]] com [[Quarkus]] e [[Maven]] no backend. Node.js e [[NestJS]] são alternativas baseadas em JavaScript ou TypeScript.

| Característica | Node.js | Java com Quarkus |
| --- | --- | --- |
| O que é | runtime de JavaScript | linguagem e runtime sobre a JVM, com framework Quarkus |
| Linguagem comum | JavaScript/TypeScript | [[Java|Java 17]] |
| Ecossistema | npm e pacotes JavaScript | [[Maven]] e bibliotecas Java |
| Modelo comum | assíncrono e orientado a eventos | threads, I/O e recursos da JVM, conforme a aplicação |
| Uso | APIs, ferramentas e serviços | APIs e serviços backend |

As duas opções podem construir backend. A escolha depende da equipe, da experiência, dos requisitos de desempenho, da integração com bibliotecas e da stack já adotada.

## Docker e execução

Uma aplicação Node.js pode ser executada em [[Docker]]:

```dockerfile
FROM node:<versao-lts>

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

CMD ["npm", "run", "start"]
```

- a imagem usa uma versão LTS fixada pela equipe;
- o `package*.json` é copiado antes do restante para aproveitar o cache de dependências;
- `npm ci` instala as versões do lockfile;
- o comando final inicia a aplicação.

Em produção, também configure usuário não-root, limites de recursos, logs, health check, rede, secrets e encerramento correto do processo.

## Versões e LTS

Node.js possui linhas de versão com ciclos diferentes. Para produção, prefira uma versão **LTS** (*Long-Term Support*) que ainda receba correções, em vez de escolher automaticamente a versão Current ou usar uma tag indefinida.

Fixe a versão no ambiente local, no Docker e no CI. A [página oficial de downloads](https://nodejs.org/en/download) mostra as linhas atuais, LTS e encerradas.

## Segurança e boas práticas

- use uma versão LTS suportada;
- mantenha `package.json` e `package-lock.json` versionados;
- use `npm ci` em ambientes automatizados;
- revise dependências antes de instalar pacotes;
- atualize dependências com testes e revisão de compatibilidade;
- execute `npm audit` e avalie os resultados, sem aceitar correções automaticamente sem revisão;
- não execute scripts de pacotes desconhecidos sem entender o risco;
- mantenha segredos em variáveis de ambiente ou em um gerenciador de segredos;
- valide entradas recebidas por HTTP, arquivos e mensagens;
- configure timeouts para chamadas externas;
- não bloqueie o event loop com cálculos ou arquivos muito grandes;
- limite tamanho de payloads e uploads;
- trate erros sem expor stack traces ou credenciais;
- registre logs úteis, mas masque dados pessoais e tokens;
- use [[Testes]] para verificar regras e integrações;
- monitore memória, CPU, event loop, conexões, latência e reinícios.

## Resumo

**Node.js é um runtime que executa JavaScript fora do navegador. Seu modelo assíncrono e orientado a eventos é adequado para APIs, servidores e ferramentas, mas a aplicação ainda precisa de framework, banco, segurança, testes e observabilidade.**

## Referências oficiais

- [Sobre o Node.js](https://nodejs.org/en/about);
- [Documentação da API](https://nodejs.org/api/);
- [Módulos de pacote](https://nodejs.org/api/packages.html);
- [Servidor HTTP](https://nodejs.org/api/http.html);
- [Variáveis de ambiente](https://nodejs.org/api/environment_variables.html);
- [Test runner](https://nodejs.org/api/test.html).
