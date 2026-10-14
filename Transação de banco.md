# O que é uma transação de banco?

Uma **transação de banco de dados** (*database transaction*) é um grupo de operações tratado como uma unidade. O banco confirma o conjunto quando tudo dá certo ou desfaz as alterações da transação quando ocorre uma falha antes da confirmação.

Uma analogia é uma transferência bancária: retirar dinheiro de uma conta e depositá-lo em outra são duas partes da mesma operação. Se o débito acontecer e o crédito falhar, o sistema não deve deixar o dinheiro desaparecer. A transação permite confirmar as duas alterações juntas ou cancelar o conjunto.

Transações são um recurso de [[Banco de dados|bancos de dados]] e são especialmente importantes em operações que alteram mais de um dado relacionado.

## `BEGIN`, `COMMIT` e `ROLLBACK`

Em SQL, o fluxo manual mais comum é:

- `BEGIN`: inicia a transação;
- `COMMIT`: confirma as alterações;
- `ROLLBACK`: cancela as alterações ainda não confirmadas.

Exemplo em [[PostgreSQL]]:

```sql
BEGIN;

UPDATE contas
SET saldo = saldo - 100.00
WHERE id = 1 AND saldo >= 100.00;

UPDATE contas
SET saldo = saldo + 100.00
WHERE id = 2;

COMMIT;
```

O exemplo ilustra as duas alterações dentro de uma transação, mas a aplicação ainda precisa verificar quantas linhas cada `UPDATE` modificou. Se a conta de origem não tiver saldo suficiente, ou a conta de destino não existir, a operação deve executar `ROLLBACK` em vez de confirmar apenas parte do fluxo.

Uma versão conceitual com verificações no código seria:

```text
BEGIN
debitar 100 da conta de origem, somente se houver saldo
se nenhuma conta foi debitada: ROLLBACK e informar o motivo

creditar 100 na conta de destino
se nenhuma conta foi creditada: ROLLBACK e informar o motivo

COMMIT
```

Se uma transação termina sem `COMMIT`, a conexão precisa ser tratada corretamente. Bibliotecas e frameworks geralmente fazem commit ou rollback automaticamente conforme o resultado do método; conheça o comportamento da ferramenta utilizada.

Cada instrução SQL também é executada dentro de uma transação implícita quando não existe um bloco explícito. Um bloco explícito permite agrupar várias instruções. A documentação do PostgreSQL explica [transações e blocos de transação](https://www.postgresql.org/docs/18/tutorial-transactions.html).

## Propriedades [[ACID]]

As propriedades clássicas das transações são resumidas pela sigla **ACID**:

- **Atomicidade (*atomicity*):** o grupo é confirmado como um todo ou cancelado como um todo;
- **Consistência (*consistency*):** as regras definidas para os dados continuam válidas antes e depois da transação;
- **Isolamento (*isolation*):** transações simultâneas não devem produzir resultados incompatíveis com o nível de isolamento escolhido;
- **Durabilidade (*durability*):** depois que o banco confirma a transação, as alterações são preservadas conforme as garantias configuradas no banco.

Uma transação não inventa as regras de consistência por conta própria. Restrições como `NOT NULL`, `CHECK`, chaves estrangeiras e regras de negócio ajudam a definir quais estados são válidos.

## O que acontece quando falha?

Se uma instrução falhar, a aplicação normalmente deve encerrar a unidade de trabalho com rollback. Em PostgreSQL, depois de um erro dentro de um bloco, a transação fica em estado de falha e precisa ser revertida antes de continuar, salvo se o erro estiver isolado por um savepoint.

Exemplo:

```sql
BEGIN;

INSERT INTO pedidos (id, cliente_id)
VALUES (1001, 42);

-- Se uma instrução seguinte falhar:
ROLLBACK;
```

O rollback cancela as alterações não confirmadas desse bloco. Ele não desfaz uma transação que já recebeu `COMMIT` e não reverte automaticamente ações executadas fora do banco.

## Savepoints

Um **savepoint** marca um ponto interno ao qual é possível voltar sem cancelar necessariamente toda a transação:

```sql
BEGIN;

UPDATE pedidos
SET observacao = 'Revisado'
WHERE id = 1001;

SAVEPOINT antes_do_ajuste;

UPDATE estoque
SET quantidade = quantidade - 1
WHERE produto_id = 77;

-- Se for preciso desfazer só o ajuste posterior:
ROLLBACK TO SAVEPOINT antes_do_ajuste;

COMMIT;
```

Savepoints podem ser úteis para tratar uma etapa opcional, mas não devem esconder erros que deveriam cancelar a operação inteira.

## Isolamento e concorrência

Várias transações podem executar ao mesmo tempo. O isolamento determina o que uma transação pode observar enquanto outras leem ou alteram os mesmos dados.

Algumas situações que podem surgir em concorrência:

- **leitura suja (*dirty read*):** ler uma alteração que ainda não foi confirmada;
- **leitura não repetível (*non-repeatable read*):** ler uma linha e, mais tarde na mesma transação, encontrar outro valor confirmado por outra transação;
- **leitura fantasma (*phantom read*):** repetir uma consulta por condição e encontrar um conjunto diferente de linhas;
- **anomalia de serialização (*serialization anomaly*):** o resultado combinado não corresponde a nenhuma execução sequencial válida.

O [[PostgreSQL]] oferece quatro nomes padrão de isolamento, mas suas garantias concretas dependem do banco:

| Nível | Ideia geral | Particularidade no PostgreSQL |
| --- | --- | --- |
| `Read Uncommitted` | permite as garantias mais fracas | comporta-se como `Read Committed` |
| `Read Committed` | cada instrução vê dados já confirmados antes de começar | é o padrão; instruções sucessivas podem observar commits diferentes |
| `Repeatable Read` | mantém uma visão estável dos dados durante a transação | evita leituras não repetíveis e fantasmas, mas ainda pode ocorrer anomalia de serialização |
| `Serializable` | transações confirmadas devem equivaler a alguma execução uma por vez | o banco pode abortar uma transação conflitante; a aplicação deve estar pronta para tentar novamente |

Os nomes vêm do padrão SQL, mas os detalhes variam entre bancos. No PostgreSQL, `Serializable` pode retornar um erro de falha de serialização; a aplicação deve repetir a transação inteira com limite e política de espera. Veja a [documentação oficial sobre isolamento](https://www.postgresql.org/docs/current/transaction-iso.html).

Escolher o nível mais alto para tudo não é sempre necessário. Níveis mais estritos podem aumentar conflitos e exigir novas tentativas. Escolha conforme as regras que precisam ser protegidas e teste operações concorrentes.

## Bloqueios e deadlocks

Para coordenar alterações simultâneas, o banco pode manter **bloqueios (*locks*)** sobre linhas ou outros recursos. Uma transação pode esperar até outra liberar um bloqueio.

Um **deadlock** acontece quando transações esperam umas pelas outras em um ciclo. Por exemplo, a primeira transação bloqueia a conta A e espera a conta B, enquanto a segunda bloqueia B e espera A. O banco costuma detectar o ciclo e abortar uma delas.

Boas práticas para reduzir problemas:

- manter transações curtas;
- acessar recursos em uma ordem consistente;
- criar índices adequados para evitar buscas que bloqueiam trabalho demais;
- tratar erros de deadlock e serialização com tentativas limitadas;
- não fazer chamadas lentas de rede enquanto mantém bloqueios desnecessariamente.

## Transação em uma aplicação Java

Na aplicação, a fronteira da transação costuma envolver o caso de uso que precisa ser atômico. Em um projeto com [[Java]] e [[Quarkus]], por exemplo:

```java
@ApplicationScoped
public class TransferenciaService {

    @Transactional
    public void transferir(long origemId, long destinoId, BigDecimal valor) {
        validarValor(valor);
        debitar(origemId, valor);
        creditar(destinoId, valor);
    }
}
```

O exemplo é ilustrativo: as funções `debitar` e `creditar` devem verificar as regras e sinalizar falhas para que a transação seja revertida. A configuração concreta depende do provedor de persistência e do framework.

O padrão [[ORM]] pode abrir e participar de transações, mas não elimina a necessidade de entender onde a transação começa, quando termina e como as exceções são tratadas.

## O que uma transação não faz?

Uma transação local do banco coordena operações naquele banco. Ela não desfaz automaticamente:

- um e-mail já enviado;
- uma mensagem já publicada em um broker;
- um pagamento processado por outro serviço;
- um arquivo enviado para armazenamento externo;
- uma chamada HTTP concluída com sucesso.

Por exemplo, se a aplicação publicar uma mensagem e depois a gravação do pedido falhar, a transação do PostgreSQL não consegue apagar a mensagem do [[RabbitMQ]]. Esse problema de duas escritas é tratado por padrões como o [[Outbox Pattern]].

Em processos distribuídos maiores, pode ser necessário usar [[Filas]], operações [[Idempotência|idempotentes]] e padrões de compensação. Uma transação de banco não equivale a uma transação global entre todos os serviços.

## Boas práticas

- agrupar somente as operações que precisam ter sucesso ou falha em conjunto;
- manter a transação curta para reduzir bloqueios e conflitos;
- validar regras e verificar o resultado das alterações antes do commit;
- deixar o banco reforçar invariantes com constraints;
- garantir rollback em caminhos de erro;
- definir limites e espera progressiva para tentativas após conflitos transitórios;
- tornar seguro repetir uma operação quando a aplicação pode tentar novamente;
- não executar chamadas externas demoradas enquanto a transação está aberta sem uma necessidade clara;
- testar falhas entre etapas e acessos concorrentes em ambiente de teste;
- não confundir rollback com recuperação de uma transação já confirmada;
- monitorar transações longas, deadlocks, erros e conexões ocupadas.

## Resumo

> **Uma transação de banco agrupa operações relacionadas para que sejam confirmadas juntas ou canceladas juntas.**

ACID descreve propriedades importantes, enquanto isolamento e bloqueios ajudam a coordenar transações concorrentes. Para integrações com sistemas externos, são necessários mecanismos adicionais, como o [[Outbox Pattern]].

## Referências

- [PostgreSQL 18: Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html)
- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
