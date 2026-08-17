# PostgreSQL

**PostgreSQL** é um sistema gerenciador de banco de dados (*database management system* ou DBMS) relacional, gratuito e de código aberto (*open source*).

Ele guarda dados em tabelas, entende SQL e oferece recursos para consultar informações, controlar permissões, garantir transações e manter os dados consistentes.

Uma analogia é pensar em um gerente de arquivo:

- o PostgreSQL organiza as estantes e fichas;
- o [[Backend]] faz os pedidos ao gerente;
- o SQL é a linguagem usada para fazer esses pedidos;
- as transações garantem que uma operação importante seja concluída por inteiro ou desfeita por inteiro.

## PostgreSQL na stack deste projeto

Nesta [[Stack]], o PostgreSQL será o banco de dados da aplicação. O [[Backend]], construído com [[Java|Java 17]] e [[Quarkus]], poderá enviar consultas SQL ou usar uma biblioteca/[[Framework]] para acessar os dados.

Uma aplicação normalmente não conversa com o PostgreSQL diretamente a partir do [[Frontend]]. O caminho mais seguro é:

1. o frontend envia uma [[Requisição]];
2. o backend valida a requisição;
3. o backend consulta ou altera o PostgreSQL;
4. o backend devolve uma resposta ao frontend.

## Estrutura do PostgreSQL

O PostgreSQL possui níveis de organização:

- **servidor:** processo do PostgreSQL que aceita conexões;
- **banco de dados (*database*):** espaço separado para um conjunto de informações;
- **schema:** agrupamento lógico de tabelas e outros objetos;
- **tabela (*table*):** estrutura com colunas e linhas;
- **coluna (*column*):** tipo de informação guardada;
- **linha ou registro (*row*):** uma ocorrência dos dados;
- **índice (*index*):** estrutura que acelera algumas buscas;
- **função e procedure:** código que pode executar operações dentro do banco.

## Exemplo de tabela

```sql
CREATE TABLE produtos (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    preco NUMERIC(10, 2) NOT NULL CHECK (preco >= 0),
    criado_em TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO produtos (nome, preco)
VALUES ('Caderno', 25.90);

SELECT id, nome, preco
FROM produtos
WHERE preco > 20
ORDER BY nome;
```

O exemplo cria uma tabela, impede nome vazio, impede preço negativo, registra a data automaticamente, insere um produto e consulta produtos acima de 20.

## Exemplo com Docker Compose

Para estudar localmente, o PostgreSQL pode ser executado em um container. Um exemplo simples é:

```yaml
services:
  banco:
    image: postgres:17
    environment:
      POSTGRES_DB: estudo
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "127.0.0.1:5432:5432"
    volumes:
      - postgres_dados:/var/lib/postgresql/data

volumes:
  postgres_dados:
```

Esse arquivo:

- usa uma imagem de PostgreSQL com versão definida;
- cria um banco chamado `estudo`;
- usa o usuário `app`;
- lê a senha de uma variável de ambiente, em vez de deixá-la escrita no arquivo;
- expõe a porta somente no computador local;
- mantém os dados em um volume, para que não sejam apagados quando o container for recriado.

Em produção, o banco não deve ficar exposto publicamente sem necessidade. O ideal é permitir acesso apenas ao [[Backend]] e manter backups fora do container e da máquina principal.

## PostgreSQL em uma VPS

Em uma [[VPS]], a equipe pode instalar e administrar o PostgreSQL diretamente. Isso oferece bastante controle, mas também exige cuidar de:

- atualizações;
- usuários e permissões;
- firewall e rede;
- backups e restauração;
- monitoramento;
- espaço em disco;
- replicação e alta disponibilidade, quando necessárias.

Ter o PostgreSQL em um container facilita a instalação e a repetição do ambiente, mas não substitui backup, segurança e monitoramento.

## Equivalente na AWS

Na AWS, o serviço mais próximo de administrar PostgreSQL é o **Amazon RDS for PostgreSQL**. Ele é um banco gerenciado: a AWS cuida de parte da infraestrutura, manutenção, backups configuráveis e opções de alta disponibilidade.

As principais diferenças são:

- PostgreSQL em uma VPS dá mais controle sobre o sistema operacional e a configuração, mas exige mais trabalho de administração;
- RDS reduz o trabalho operacional, mas oferece menos acesso ao sistema e pode ter custo maior;
- em uma VPS, a equipe é responsável por configurar backups e recuperação;
- no RDS, muitos recursos de manutenção e backup são oferecidos pelo serviço, mas ainda é necessário configurar, monitorar e testar a restauração;
- os dois usam PostgreSQL e podem ser acessados pelo backend por uma conexão de banco.

O RDS não é simplesmente um PostgreSQL instalado em uma VPS: é um serviço gerenciado com regras, limites e recursos próprios da AWS.

## Boas práticas

- fixar uma versão validada da imagem ou do servidor, evitando mudanças inesperadas;
- guardar senhas em variáveis de ambiente ou em um gerenciador de segredos;
- não expor a porta `5432` diretamente para toda a internet;
- criar usuários diferentes para aplicação, administração e migrações;
- dar à aplicação somente as permissões necessárias;
- usar migrações para controlar a evolução das tabelas;
- criar índices com base em consultas reais, verificando se melhoram o desempenho;
- fazer backups automáticos e testar a restauração em um ambiente separado;
- monitorar espaço, conexões, erros e consultas lentas;
- usar consultas parametrizadas e nunca montar SQL juntando diretamente valores de uma [[Requisição]].

## Resumo

> **PostgreSQL é um banco de dados relacional de código aberto, usado para guardar e consultar informações com segurança e consistência.**

Ele pode ser executado localmente, em uma [[VPS]] ou como serviço gerenciado no Amazon RDS.
