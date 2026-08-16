# Banco de dados

Um **banco de dados** (*database*) é um lugar organizado para guardar, consultar e alterar informações de um sistema.

Uma analogia é imaginar um arquivo de biblioteca:

- as estantes são as tabelas;
- as fichas são os registros;
- os campos da ficha são as colunas;
- o catálogo ajuda a encontrar uma informação rapidamente;
- as regras do arquivo impedem dados inválidos ou duplicados.

O banco de dados trabalha junto com o [[Backend]]. Quando uma pessoa faz uma ação no [[Frontend]], o backend pode consultar ou alterar informações no banco.

## O que um banco de dados faz?

Um banco de dados pode:

- guardar informações de forma organizada;
- buscar dados usando filtros;
- criar, alterar e excluir registros;
- relacionar informações de tabelas diferentes;
- controlar quem pode ler ou alterar os dados;
- manter regras para evitar informações inválidas;
- realizar cópias de segurança (*backups*);
- permitir que várias pessoas usem o sistema ao mesmo tempo.

## Banco de dados relacional

Em um banco relacional (*relational database*), os dados ficam em tabelas formadas por linhas e colunas.

Por exemplo, uma tabela `usuarios` poderia ser:

| id | nome | email |
|---:|---|---|
| 1 | Ana | ana@example.com |
| 2 | Bruno | bruno@example.com |

O campo `id` identifica cada registro. Uma tabela de pedidos poderia guardar um `usuario_id` para indicar quem fez cada pedido. Essa ligação é chamada de chave estrangeira (*foreign key*).

O [[PostgreSQL]] é um banco de dados relacional.

## SQL

Bancos relacionais normalmente usam SQL (*Structured Query Language*) para conversar com os dados.

Exemplo:

```sql
CREATE TABLE usuarios (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE
);

INSERT INTO usuarios (nome, email)
VALUES ('Ana', 'ana@example.com');

SELECT id, nome, email
FROM usuarios
WHERE email = 'ana@example.com';
```

Nesse exemplo:

- `CREATE TABLE` cria a tabela;
- `PRIMARY KEY` identifica cada registro;
- `NOT NULL` exige o preenchimento do campo;
- `UNIQUE` impede e-mails repetidos;
- `INSERT` adiciona um registro;
- `SELECT` consulta informações.

As operações de criar, consultar, alterar e excluir costumam ser resumidas pela sigla **CRUD**: *Create, Read, Update, Delete*.

## Transações

Uma transação (*transaction*) agrupa várias operações que devem ser tratadas como uma unidade.

Imagine uma transferência bancária: o sistema precisa retirar o dinheiro de uma conta e adicionar à outra. Se apenas uma parte acontecer, os dados ficam errados. A transação permite confirmar tudo junto ou desfazer tudo se houver um problema.

## Banco de dados e aplicação

O [[Backend]] não deve permitir que qualquer pessoa se conecte diretamente ao banco. A aplicação recebe a [[Requisição]], verifica a identidade e a permissão, aplica as regras e só então consulta os dados necessários.

Essa separação protege o banco e evita que a interface tenha acesso direto a informações sensíveis.

## Boas práticas

- modelar as tabelas antes de começar, evitando dados repetidos e confusos;
- usar chaves primárias e estrangeiras, mantendo os relacionamentos corretos;
- aplicar restrições como `NOT NULL`, `UNIQUE` e `CHECK`, fazendo o próprio banco ajudar a proteger os dados;
- usar consultas parametrizadas, evitando ataques de injeção SQL (*SQL injection*);
- criar índices apenas quando fizer sentido, melhorando buscas sem ocupar recursos desnecessariamente;
- usar migrações (*migrations*) para registrar mudanças na estrutura do banco;
- fazer backups automáticos e testar a restauração;
- dar a cada usuário somente as permissões necessárias, seguindo o princípio do menor privilégio;
- não guardar senhas em texto puro; armazenar apenas valores protegidos por algoritmos próprios para senhas;
- monitorar consultas lentas, espaço em disco e erros.

## Resumo

> **Banco de dados é o sistema organizado que guarda e fornece as informações usadas por uma aplicação.**

Ele é como um arquivo bem catalogado: guarda os dados, ajuda a encontrá-los e aplica regras para mantê-los corretos.
