# Operação idempotente

Uma operação é **idempotente** (*idempotent operation*) quando pode ser executada várias vezes e o efeito final é o mesmo que seria obtido executando-a uma única vez.

Em forma de ideia:

```text
aplicar(aplicar(estado)) = aplicar(estado)
```

Isso é importante porque redes falham, respostas podem ser perdidas e sistemas distribuídos costumam fazer novas tentativas (*retries*). Se a mesma mensagem chegar duas vezes, a aplicação deve evitar efeitos duplicados quando a operação precisar ser idempotente.

## Uma analogia

Pedir para uma pessoa colocar o interruptor na posição “ligado” é idempotente: repetir o pedido mantém a luz ligada.

Pedir para ela apertar um botão que adiciona R$ 10 ao saldo não é idempotente: repetir o pedido adiciona R$ 20.

O importante é o estado final, não necessariamente a resposta textual ou os eventos de observabilidade. Uma segunda tentativa de excluir um registro pode retornar `404`, por exemplo, mas o registro continua excluído.

## Exemplos simples

Idempotentes:

```text
definir status = "pago"
definir endereço = "Rua A"
apagar usuário 42
```

Não idempotentes, sem uma proteção adicional:

```text
adicionar 1 ao contador
criar um novo pagamento
enviar um e-mail
gerar um novo pedido
```

Uma operação não se torna idempotente apenas porque usa o método HTTP `POST` ou `PUT`. A idempotência depende do comportamento implementado pelo [[Backend]].

## Idempotência nos métodos HTTP

Pela semântica HTTP, alguns métodos são definidos como idempotentes:

- `GET`: consulta um recurso e não deveria alterá-lo;
- `HEAD`: consulta metadados sem o corpo da resposta;
- `OPTIONS`: consulta capacidades do recurso;
- `PUT`: substitui o recurso pela representação enviada;
- `DELETE`: remove o recurso.

`POST` geralmente não é idempotente, porque pode criar um novo recurso a cada chamada. `PATCH` pode ser idempotente ou não, dependendo da regra implementada.

Exemplo de `PUT` idempotente:

```http
PUT /usuarios/42
Content-Type: application/json

{
  "nome": "Ana",
  "ativo": true
}
```

Enviar esse mesmo pedido três vezes deve deixar o usuário com os mesmos valores, sem criar três usuários.

Exemplo de `DELETE`:

```http
DELETE /usuarios/42
```

Depois que o usuário foi removido, repetir o pedido pode retornar `404`, mas não deve remover outro usuário nem produzir um efeito destrutivo adicional.

Método idempotente não significa método seguro. `DELETE` pode ser idempotente e ainda assim destruir dados. Também pode haver efeitos auxiliares, como registros de log, métricas e auditoria, que mudam a cada chamada.

## Chave de idempotência

Para operações que normalmente usam `POST`, a aplicação pode aceitar uma **idempotency key** (chave de idempotência):

```http
POST /pagamentos
Idempotency-Key: pagamento-8f3a1c
Content-Type: application/json

{
  "pedidoId": 42,
  "valor": 150.00
}
```

O servidor deve:

1. receber a chave e validar os dados;
2. verificar se aquela chave já foi processada;
3. se já foi, devolver o mesmo resultado ou indicar que o processamento está em andamento;
4. se ainda não foi, executar a operação;
5. guardar a chave, o resultado e o estado do processamento;
6. devolver a resposta ao cliente.

Assim, se a resposta se perder e o cliente repetir a requisição com a mesma chave, o servidor não cria um segundo pagamento.

Boas regras para a chave:

- deve ser única para uma intenção de negócio;
- deve ter formato imprevisível o suficiente para não colidir;
- deve expirar depois de um período adequado;
- deve ficar associada aos mesmos parâmetros da primeira tentativa;
- se a mesma chave for usada com dados diferentes, o servidor deve rejeitar o pedido;
- não deve conter dados sensíveis desnecessários.

Uma requisição feita com [[cURL]] pode usar a mesma ideia:

```bash
curl --request POST \
  --url https://api.exemplo.com/pagamentos \
  --header 'Idempotency-Key: pagamento-8f3a1c' \
  --header 'Content-Type: application/json' \
  --data '{"pedidoId":42,"valor":150.00}'
```

## Idempotência e filas

Filas normalmente podem entregar a mesma mensagem mais de uma vez, principalmente quando trabalham com pelo menos uma entrega (*at-least-once delivery*). Por isso, um consumidor deve ser preparado para mensagens duplicadas.

Uma mensagem pode carregar um identificador único:

```json
{
  "eventId": "evento-123",
  "tipo": "PagamentoAprovado",
  "pagamentoId": 42
}
```

O worker pode guardar `eventId` em uma tabela com restrição única. Se receber o mesmo evento novamente, verifica que ele já foi processado e não repete a operação.

Outras estratégias:

- usar uma chave única no banco;
- atualizar para um estado específico, em vez de incrementar cegamente;
- salvar o evento processado e a mudança de negócio na mesma transação;
- usar um padrão outbox para coordenar banco e fila;
- separar falha temporária de falha permanente;
- enviar mensagens que excederam tentativas para uma DLQ.

Veja também [[Filas]] para entender produtores, consumidores, retries e confirmações.

## Idempotência no banco de dados

Uma operação que define um valor tende a ser idempotente:

```sql
UPDATE pedidos
SET status = 'pago'
WHERE id = 42;
```

Uma operação que incrementa um valor não é idempotente por si só:

```sql
UPDATE contas
SET saldo = saldo + 10
WHERE id = 42;
```

Se o segundo comando for repetido, o saldo aumenta novamente. Para proteger uma operação financeira, use uma identificação única da transação, restrições no banco e uma transação bem definida.

`INSERT` também pode criar duplicatas se for repetido. Uma chave única, um `UPSERT` ou uma verificação transacional pode ajudar, mas a solução deve considerar concorrência e o resultado esperado.

## O que idempotência não significa

- **Não significa que a operação não altera dados**: `DELETE` pode alterar o estado uma vez e ser idempotente nas repetições.
- **Não significa que a resposta será sempre igual**: o status pode mudar de `200` para `404` após uma exclusão.
- **Não significa atomicidade**: atomicidade é concluir tudo ou nada em uma transação.
- **Não significa exatamente uma execução física**: o servidor pode executar mais de uma tentativa, desde que o efeito de negócio não seja duplicado.
- **Não elimina a necessidade de autenticação**: quem pode repetir a operação ainda precisa ser autorizado.

## Como projetar uma operação idempotente

1. defina qual é o efeito de negócio que não pode ser repetido;
2. escolha uma identificação única para a intenção ou evento;
3. guarde o estado da operação de forma durável;
4. use restrições e transações no banco quando necessário;
5. devolva um resultado consistente nas repetições;
6. defina expiração e limpeza das chaves antigas;
7. registre tentativas sem expor tokens ou dados sensíveis;
8. documente o comportamento para quem consome a API.

## Testes

Inclua [[Testes]] que executem a mesma operação duas ou mais vezes e verifiquem:

- se o efeito final é o esperado;
- se não foram criados registros duplicados;
- se a mesma chave devolve o resultado correto;
- se uma falha antes da resposta permite uma nova tentativa segura;
- se duas requisições concorrentes não produzem um efeito duplicado;
- se uma chave usada com parâmetros diferentes é rejeitada.

## Segurança

Idempotência não substitui [[Segurança]]. A chave de idempotência não é uma senha nem prova de identidade. O servidor ainda deve validar autenticação, autorização, entrada, escopo e propriedade do recurso.

Não use IDs previsíveis para permitir acesso a dados de outra pessoa. Uma chave pode evitar duplicação, mas não deve ser tratada como permissão.

## Boas práticas

- planeje idempotência para pagamentos, pedidos, reservas, criação de recursos e tarefas com retry;
- prefira operações de definição de estado quando fizer sentido;
- use chaves únicas, transações e restrições no banco;
- mantenha o resultado associado à chave por tempo suficiente para cobrir retries;
- defina o que acontece quando a operação ainda está em processamento;
- trate concorrência, timeout e mensagens duplicadas;
- não faça retry cego de um `POST` que produz efeitos;
- documente quais métodos e endpoints são idempotentes;
- teste repetição, concorrência e falhas de rede.

**Uma operação idempotente pode ser repetida sem duplicar seu efeito final, tornando APIs, filas e sistemas distribuídos mais seguros diante de retries e falhas de comunicação.**
