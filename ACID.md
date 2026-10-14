# O que significa ACID?

**ACID** é uma sigla para quatro propriedades que ajudam a descrever garantias de uma [[Transação de banco|transação de banco de dados]]:

- **A — Atomicidade (*Atomicity*):** todas as operações da transação acontecem ou nenhuma delas é confirmada;
- **C — Consistência (*Consistency*):** uma transação válida leva o banco de um estado válido a outro estado válido, conforme as regras definidas;
- **I — Isolamento (*Isolation*):** transações simultâneas não devem interferir umas nas outras de forma proibida pelo nível de isolamento escolhido;
- **D — Durabilidade (*Durability*):** depois da confirmação, as alterações devem permanecer registradas de acordo com as garantias do banco e sua configuração.

Uma analogia é uma transferência bancária. O sistema debita uma conta e credita outra. A operação deve ser completa, manter as regras dos saldos, lidar com outras transferências simultâneas e continuar registrada depois de confirmada.

ACID descreve propriedades relacionadas; não é um comando, biblioteca ou opção única que automaticamente torna qualquer aplicação correta.

## A — Atomicidade

Atomicidade significa **tudo ou nada** para as operações dentro de uma transação.

Se a transferência debitar a conta de origem, mas falhar antes de creditar a conta de destino, a transação pode ser revertida. Assim, o débito não fica confirmado sozinho.

```sql
BEGIN;

UPDATE contas
SET saldo = saldo - 50
WHERE id = 1 AND saldo >= 50;

UPDATE contas
SET saldo = saldo + 50
WHERE id = 2;

COMMIT;
```

Esse exemplo ainda precisa que a aplicação confira se as duas atualizações afetaram uma linha. Se uma conta não existir ou o saldo for insuficiente, deve executar `ROLLBACK` em vez de confirmar uma operação incompleta.

Atomicidade vale para as operações dentro da transação do banco. Ela não desfaz automaticamente um e-mail enviado, uma chamada HTTP ou uma mensagem já publicada em um broker.

## C — Consistência

Consistência significa preservar as regras que definem quais dados e estados são válidos. A transação deve respeitar essas regras ao passar de um estado válido para outro.

O banco pode proteger algumas regras com constraints:

```sql
CREATE TABLE contas (
    id BIGINT PRIMARY KEY,
    saldo NUMERIC(12, 2) NOT NULL CHECK (saldo >= 0)
);
```

Aqui, o banco rejeita saldos negativos e valores nulos. Outras regras podem depender da aplicação, como verificar se a pessoa tem autorização para movimentar aquela conta.

Uma transação não descobre sozinha todas as regras do negócio. O sistema precisa defini-las e aplicá-las no local adequado, combinando validações no [[Backend]] e restrições no [[Banco de dados]].

## I — Isolamento

Isolamento trata do que acontece quando várias transações trabalham ao mesmo tempo. Ele evita que a concorrência produza resultados incompatíveis com as garantias do banco.

Imagine duas compras tentando reservar a última unidade de um produto. Se ambas lerem “estoque = 1” e diminuírem o valor sem coordenação, o sistema pode aceitar duas reservas. A transação precisa usar as garantias de isolamento, bloqueios ou atualizações condicionais para preservar a regra.

Os bancos oferecem níveis diferentes de isolamento. Um nível mais forte pode reduzir certas anomalias, mas também pode causar mais conflitos ou abortos que a aplicação precisa tratar. No [[PostgreSQL]], por exemplo, `Read Committed` é o padrão e `Serializable` pode abortar uma transação para impedir um resultado incompatível; em certos casos, a aplicação deve repetir a transação inteira.

Os nomes e detalhes variam entre bancos. A nota [[Transação de banco]] explica níveis de isolamento, bloqueios e deadlocks com mais detalhes.

## D — Durabilidade

Durabilidade significa que, depois de um `COMMIT` confirmado, as alterações não devem desaparecer por uma falha comum do processo ou do sistema, conforme a configuração e as garantias do banco.

Durabilidade não substitui backup. Um backup ajuda a recuperar dados apagados por engano, corrupção lógica ou incidentes maiores; uma transação confirmada pode preservar fielmente até uma alteração errada. Por isso, faça backups e teste a restauração.

## O que ACID não garante sozinho?

ACID não garante que:

- a regra de negócio esteja correta;
- o usuário esteja autorizado a realizar a operação;
- dados externos estejam atualizados;
- uma mensagem enviada a outro sistema seja processada uma só vez;
- o sistema inteiro, com vários serviços, participe de uma única transação local;
- exista backup ou recuperação de qualquer tipo de erro.

Por exemplo, uma transação do [[PostgreSQL]] pode gravar um pedido e finalizar com `COMMIT`, mas não consegue confirmar atomicamente uma mensagem em um [[RabbitMQ]]. Para coordenar a gravação local e a publicação de eventos, o [[Outbox Pattern]] registra o evento junto dos dados e publica depois.

## Como usar essas propriedades

Ao implementar uma operação importante, pergunte:

1. Quais alterações precisam ser confirmadas juntas?
2. Quais regras de validade devem ser mantidas?
3. O que acontece se duas solicitações alterarem os mesmos dados?
4. Como a aplicação reage a conflitos ou falhas antes do commit?
5. Quais garantias de recuperação e backup são necessárias?
6. A operação chama serviços externos que precisam de outro padrão?

Depois, escreva [[Testes]] para caminhos de sucesso, falha e concorrência. Os testes devem verificar o estado final do banco, não apenas se uma função foi chamada.

## Boas práticas

- agrupe em uma transação somente as mudanças que precisam ser atômicas;
- defina invariantes e aplique constraints no banco quando apropriado;
- escolha o nível de isolamento conforme o risco e as regras do caso de uso;
- trate erros de deadlock e serialização com tentativas limitadas;
- mantenha as transações curtas;
- não confunda commit com backup;
- não suponha que uma transação do banco inclui serviços externos;
- use [[Idempotência|operações idempotentes]] quando uma solicitação puder ser repetida;
- monitore transações longas, falhas e conflitos.

## Resumo

> **ACID resume quatro garantias importantes das transações: atomicidade, consistência, isolamento e durabilidade.**

Essas propriedades ajudam o banco a manter dados confiáveis, mas dependem de regras bem definidas, uso correto das transações, configuração adequada e tratamento dos sistemas externos.
