# Prisma

**Prisma** é um ORM (*Object-Relational Mapper*) e toolkit de acesso a bancos de dados para [[Node.js]] e [[TypeScript]]. Ele ajuda a definir o modelo dos dados, gerar um cliente de consultas com tipos e controlar alterações no banco.

Uma analogia: o banco de dados é uma biblioteca, e o Prisma é um bibliotecário que conhece a organização das estantes. Em vez de cada parte do programa escrever consultas sem padrão, o bibliotecário oferece uma forma organizada de encontrar, criar e alterar registros.

Prisma não é um banco de dados. Ele trabalha junto com soluções como [[PostgreSQL]], MySQL, SQLite, SQL Server, CockroachDB e [[MongoDB]]. A [documentação oficial de bancos suportados](https://www.prisma.io/docs/orm/core-concepts/supported-databases) deve ser consultada porque recursos e versões variam entre os bancos.

## O que é um ORM?

Um ORM cria uma ponte entre:

- objetos e classes do código;
- tabelas, colunas e relações de um banco relacional;
- ou documentos e coleções, no caso de um banco como MongoDB.

Sem ORM, a aplicação pode escrever SQL ou comandos específicos diretamente. Com ORM, ela pode trabalhar com modelos e métodos do código, enquanto a ferramenta gera ou executa as consultas necessárias.

ORM não elimina a necessidade de conhecer o banco. É importante entender SQL, índices, transações, relacionamentos, limites de conexão e o custo das consultas.

## Principais partes do Prisma

### Prisma Schema

O arquivo `schema.prisma` descreve a conexão, o gerador e os modelos da aplicação. Ele funciona como uma planta dos dados.

Exemplo para PostgreSQL:

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model Usuario {
  id      Int      @id @default(autoincrement())
  email   String   @unique
  nome    String
  pedidos Pedido[]
}

model Pedido {
  id        Int      @id @default(autoincrement())
  valor     Decimal
  status    String   @default("novo")
  usuarioId Int
  usuario   Usuario  @relation(fields: [usuarioId], references: [id])
}
```

Esse modelo descreve dois tipos de dados e uma relação: um usuário pode possuir vários pedidos, e cada pedido pertence a um usuário.

Nas versões atuais, a conexão pode ser configurada em `prisma.config.ts`:

```typescript
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

A variável `DATABASE_URL` deve ficar em um arquivo de ambiente local ou no gerenciador de segredos do ambiente. Ela não deve ser publicada no Git.

### Prisma Client

O Prisma Client é gerado a partir do schema. Ele oferece métodos tipados para consultar e alterar os modelos.

Exemplo conceitual:

```typescript
const usuarios = await prisma.usuario.findMany({
  where: {
    nome: {
      contains: "Ana",
    },
  },
  select: {
    id: true,
    nome: true,
    email: true,
  },
});
```

Nesse exemplo, o código busca usuários cujo nome contém `Ana` e seleciona apenas os campos necessários. O TypeScript consegue ajudar a identificar nomes de campos e tipos inválidos antes da aplicação ser executada.

Operações comuns:

```typescript
await prisma.usuario.create({
  data: {
    nome: "Ana",
    email: "ana@example.com",
  },
});

await prisma.usuario.findUnique({
  where: { email: "ana@example.com" },
});

await prisma.usuario.update({
  where: { email: "ana@example.com" },
  data: { nome: "Ana Silva" },
});

await prisma.usuario.delete({
  where: { email: "ana@example.com" },
});
```

Esses métodos representam operações CRUD: criar, ler, atualizar e excluir. Os filtros devem ser específicos, principalmente em atualizações e exclusões.

### Prisma Migrate

Prisma Migrate cria uma história de alterações do banco. O fluxo comum com PostgreSQL é:

1. alterar o modelo em `schema.prisma`;
2. criar uma migration durante o desenvolvimento;
3. revisar os arquivos SQL gerados;
4. aplicar a migration em outros ambientes;
5. gerar ou atualizar o Prisma Client.

Comandos de exemplo:

```bash
npx prisma migrate dev --name criar-usuarios
npx prisma generate
npx prisma migrate deploy
```

- `migrate dev` cria e aplica uma migration no ambiente de desenvolvimento;
- `generate` atualiza o cliente gerado;
- `migrate deploy` aplica migrations já versionadas em ambientes como homologação ou produção.

As migrations devem ser revisadas e versionadas junto com o código. Não use `migrate dev` diretamente em produção.

### Prisma Studio

Prisma Studio é uma interface visual que permite consultar e editar dados no navegador durante o desenvolvimento:

```bash
npx prisma studio
```

É útil para estudar o modelo e investigar dados locais. Deve ser protegido e usado com cuidado: editar dados diretamente pode ignorar regras de negócio, auditoria e validações da aplicação.

## Prisma com banco existente

Prisma não precisa criar o banco do zero. Quando o banco já existe, é possível ler sua estrutura e gerar um schema com introspecção:

```bash
npx prisma db pull
npx prisma generate
```

Esse fluxo é útil para adotar Prisma gradualmente. Depois de importar a estrutura, revise nomes, relações e tipos antes de usar o cliente em produção.

## Prisma com NestJS

No [[NestJS]], uma prática comum é criar um `PrismaService` e injetá-lo nos services de cada módulo:

```typescript
import { Injectable } from "@nestjs/common";

@Injectable()
export class UsuariosService {
  constructor(private readonly prisma: PrismaService) {}

  listar() {
    return this.prisma.usuario.findMany();
  }
}
```

O exemplo mostra a ideia principal: o controller recebe a requisição, o service aplica a regra e o Prisma acessa o banco. O código real de `PrismaService` depende da versão do Prisma, do driver e da configuração do projeto.

Mantenha uma instância compartilhada do Prisma Client por aplicação, especialmente durante o desenvolvimento com recarga automática. Criar uma conexão nova para cada requisição pode esgotar o limite de conexões do banco.

## Relações e consultas

O Prisma permite buscar relações, mas é importante trazer apenas o que será usado:

```typescript
const pedidos = await prisma.pedido.findMany({
  where: { usuarioId: 7 },
  include: {
    usuario: {
      select: {
        id: true,
        nome: true,
      },
    },
  },
});
```

`include` solicita relações; `select` limita os campos. Essa escolha ajuda a evitar respostas grandes e consultas desnecessárias.

Para listas, use paginação, filtros e ordenação definidos. Buscar todos os registros de uma tabela pode funcionar no estudo, mas pode causar lentidão e consumo excessivo de memória em produção.

## Transações

Uma transação agrupa operações que precisam ter um resultado conjunto: todas são confirmadas ou todas são desfeitas.

```typescript
await prisma.$transaction(async (tx) => {
  const pedido = await tx.pedido.create({
    data: {
      valor: 100,
      usuarioId: 7,
    },
  });

  await tx.pedido.update({
    where: { id: pedido.id },
    data: { status: "confirmado" },
  });
});
```

Use transações para preservar invariantes importantes, mas mantenha-as curtas. Uma transação longa prende conexões e locks, podendo prejudicar outras operações.

Quando o fluxo envolve banco e mensageria, uma transação do banco não confirma automaticamente uma mensagem no broker. Para esse problema, estude [[Outbox Pattern]], [[RabbitMQ]] e [[Filas]].

## PostgreSQL e MongoDB

Com [[PostgreSQL]], Prisma trabalha com tabelas, colunas, chaves estrangeiras, constraints e migrations SQL. É uma combinação coerente quando a aplicação possui relações fortes e transações relacionais.

Com [[MongoDB]], Prisma usa modelos de documentos, mas há diferenças importantes. O fluxo de schema e consultas continua existindo, porém o suporte, os tipos e as relações não são iguais aos de um banco relacional. A documentação atual informa que, durante a transição para o Prisma ORM 7, o uso com MongoDB deve seguir a versão e as instruções específicas indicadas pela documentação.

Para MongoDB, o comando usado para aplicar alterações de schema é geralmente:

```bash
npx prisma db push
```

Não trate `db push` como substituto universal de migrations relacionais. Ele é apropriado para prototipagem e para o fluxo específico do conector MongoDB; em produção, planeje como as alterações serão revisadas, aplicadas e revertidas.

## Prisma não substitui o banco nem o backend

O fluxo completo continua tendo várias partes:

```text
[[Frontend]] -> [[Requisição]] -> [[Backend]] -> Prisma -> banco de dados
```

O Prisma não decide:

- quem tem autorização para executar uma operação;
- qual regra de negócio deve ser aplicada;
- se os dados recebidos são aceitáveis para o domínio;
- como uma fila ou evento será processado;
- quais índices e limites o banco precisa;
- como o sistema fará auditoria e recuperação.

Ele é uma ferramenta de acesso e evolução do banco, não um substituto para a arquitetura da aplicação.

## Prisma e Quarkus

A stack principal deste repositório usa [[Java|Java 17]] com [[Quarkus]] e [[Maven]]. Prisma pertence principalmente ao ecossistema [[Node.js]] e [[TypeScript]], portanto não é a ferramenta padrão dessa stack.

No ecossistema Java, o acesso ao banco costuma usar outras soluções, como JDBC, JPA ou Hibernate. A ideia geral de mapear modelos para dados pode ser parecida, mas as APIs e os fluxos são diferentes.

## Segurança e boas práticas

- mantenha `DATABASE_URL` e credenciais fora do código e do Git;
- use um usuário do banco com apenas as permissões necessárias;
- valide entradas no [[Backend]] ou em DTOs antes de consultar o banco;
- não confunda tipos do [[TypeScript]] com validação de dados recebidos em tempo de execução;
- use `select` para evitar retornar campos sensíveis ou desnecessários;
- evite expor mensagens internas e consultas nos erros da API;
- prefira os métodos tipados do Prisma e tenha cuidado ao usar SQL bruto;
- se usar SQL bruto, passe parâmetros de forma segura e nunca concatene entrada do usuário;
- crie índices para consultas importantes e confirme seu efeito com análise do banco;
- use paginação em listas que podem crescer;
- mantenha migrations versionadas e revise alterações destrutivas;
- faça backup e teste restauração independentemente do ORM usado;
- monitore conexões, consultas lentas, locks, erros e tempo de resposta;
- teste regras de negócio e transações com [[Testes]].

## Vantagens e limitações

### Vantagens

- consultas com tipos gerados;
- schema central para comunicar a forma dos dados;
- migrations integradas para bancos relacionais;
- cliente com API consistente;
- Prisma Studio para inspeção local;
- boa integração com aplicações [[Node.js]], [[TypeScript]] e [[NestJS]].

### Limitações

- o código gerado não elimina o custo real das consultas;
- abstrações podem esconder detalhes importantes do banco;
- recursos avançados do banco podem exigir SQL ou configuração específica;
- migrations mal revisadas podem perder ou bloquear dados;
- o comportamento e os recursos variam por banco e versão;
- usar ORM não substitui modelagem, índices, segurança e observabilidade.

## Resumo

**Prisma é um toolkit de ORM para [[Node.js]] e [[TypeScript]] que usa um schema para gerar um cliente tipado, controlar migrations e facilitar a inspeção dos dados. Ele simplifica o acesso ao banco, mas não substitui o conhecimento de [[PostgreSQL]], [[MongoDB]], SQL, modelagem e segurança.**

## Referências oficiais

- [Visão geral do Prisma ORM](https://docs.prisma.io/docs/orm);
- [Schema do Prisma](https://docs.prisma.io/docs/orm/prisma-schema/overview);
- [Bancos suportados](https://www.prisma.io/docs/orm/core-concepts/supported-databases);
- [Prisma Migrate](https://www.prisma.io/docs/orm/prisma-migrate);
- [Prisma Studio](https://www.prisma.io/docs/orm/v6/tools/prisma-studio).
