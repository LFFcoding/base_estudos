# O que é uma chave de idempotência?

Uma **chave de idempotência** (*idempotency key*) é um identificador único enviado com uma operação para que o servidor reconheça tentativas repetidas da mesma ação.

Ela é muito útil quando uma [[Requisição]] pode ser repetida por causa de timeout, falha de rede, clique duplo ou retry automático. O cliente envia a mesma chave novamente e o servidor entende que não deve criar um novo efeito.

Uma analogia é o protocolo de retirada de uma encomenda: você recebe um código único. Se pedir confirmação duas vezes usando o mesmo código, o atendente consulta o registro original em vez de entregar a encomenda novamente.

## Por que ela é necessária?

Imagine uma pessoa clicando em “Pagar” e a conexão caindo logo depois. O pagamento pode ter sido aprovado, mas o cliente não recebeu a resposta.

Sem uma chave:

```text
tentativa 1 -> pagamento criado
cliente não recebe a resposta
tentativa 2 -> outro pagamento pode ser criado
```

Com uma chave:

```text
tentativa 1 -> pagamento criado com a chave K
cliente não recebe a resposta
tentativa 2 -> chave K já existe -> devolve o resultado original
```

O objetivo é impedir que a mesma intenção do cliente produza efeitos duplicados.

## Exemplo em uma requisição HTTP

A chave costuma ser enviada em um header:

```http
POST /pagamentos
Idempotency-Key: 4f7c9c1e-8d4e-4b7a-a9f0-3b2f8e0b1a22
Content-Type: application/json

{
  "pedidoId": 42,
  "valor": 99.90
}
```

O valor deve ser novo para cada operação lógica. Se a mesma operação precisar ser tentada novamente, o cliente deve reutilizar a mesma chave.

Um exemplo com [[cURL]]:

```bash
curl -X POST https://api.exemplo.com/pagamentos \
  -H "Idempotency-Key: 4f7c9c1e-8d4e-4b7a-a9f0-3b2f8e0b1a22" \
  -H "Content-Type: application/json" \
  -d '{"pedidoId":42,"valor":99.90}'
```

Se houver timeout, repita a chamada usando exatamente a mesma `Idempotency-Key`, e não uma chave nova.

## A chave não torna qualquer operação idempotente

A chave é um mecanismo para o servidor controlar repetições. Ela não corrige automaticamente uma regra de negócio mal projetada.

Uma operação é idempotente quando executá-la uma ou várias vezes produz o mesmo efeito final. Isso é explicado em [[Idempotência]].

Por exemplo, `PUT /usuarios/42` pode definir o mesmo e-mail várias vezes. Já `POST /pagamentos` normalmente cria um efeito e precisa de uma chave para que retries não causem outro pagamento.

## Responsabilidade do cliente

O cliente deve:

1. gerar uma chave com boa aleatoriedade, normalmente um UUID;
2. associar a chave a uma única intenção de negócio;
3. reutilizar a chave quando repetir a mesma tentativa;
4. criar outra chave para uma nova operação;
5. guardar a chave durante o período em que pode precisar fazer retry;
6. tratar respostas de timeout sem assumir automaticamente que a operação falhou.

Não gere uma chave nova a cada retry. Isso impede o servidor de reconhecer que as tentativas pertencem à mesma operação.

## Responsabilidade do servidor

O servidor deve:

1. receber e validar a chave;
2. definir o escopo da chave;
3. calcular uma impressão (*fingerprint*) da requisição, quando necessário;
4. reservar a chave de forma atômica;
5. executar a operação;
6. guardar o resultado e o status;
7. devolver o mesmo resultado em uma repetição válida;
8. tratar chamadas concorrentes com a mesma chave.

Uma implementação que apenas verifica “se existe” e depois insere pode falhar:

```text
requisição A verifica -> não existe
requisição B verifica -> não existe
requisição A cria
requisição B cria
```

A reserva precisa usar uma restrição única, uma transação ou uma operação atômica do armazenamento.

## Fluxo do servidor

Um fluxo comum é:

```text
receber chave
    ↓
validar formato e tamanho
    ↓
procurar (escopo, chave)
    ├── não existe -> reservar e processar
    ├── existe e concluída -> devolver resposta salva
    ├── existe e em processamento -> aguardar ou informar conflito
    └── mesma chave com outro payload -> rejeitar
```

### Primeira chamada

O servidor registra que a chave está em processamento e executa a operação.

### Repetição depois de concluída

O servidor localiza a chave, verifica que o payload é compatível e devolve o status e o corpo armazenados.

Dependendo do contrato da API, a segunda resposta pode ser `201 Created` novamente ou outro status documentado. O mais importante é que não crie um novo recurso ou efeito.

### Repetição enquanto processa

Duas requisições podem chegar ao mesmo tempo. O servidor precisa definir uma política:

- esperar um curto período pelo resultado;
- responder `409 Conflict` informando que a chave está em processamento;
- responder `202 Accepted` se o processamento for assíncrono;
- consultar o status por outro endpoint.

Não deixe uma requisição esperando indefinidamente.

### Mesma chave com dados diferentes

Se a chave foi usada com um corpo e depois reaparecer com outro, rejeite a chamada:

```text
chave K + pedidoId 42 + valor 99,90 -> aceita
chave K + pedidoId 43 + valor 50,00 -> rejeita
```

Um `409 Conflict` é uma opção comum, mas o contrato da API deve definir o status e o corpo de erro.

## Fingerprint da requisição

O **fingerprint** é uma representação usada para verificar se duas chamadas com a mesma chave têm os mesmos dados importantes.

Ele pode considerar:

- método HTTP;
- rota;
- usuário ou tenant;
- corpo normalizado;
- campos relevantes para o efeito.

Não basta comparar uma string do JSON sem decidir se a ordem dos campos altera ou não o significado. O servidor deve normalizar e validar o corpo antes de calcular a impressão.

Uma alternativa é guardar um hash, como SHA-256, do conteúdo relevante. O hash ajuda a comparar dados, mas não substitui a validação da requisição nem deve ser tratado como autenticação.

## Escopo da chave

Uma chave não deve ser global sem necessidade. O registro pode ser identificado por uma combinação como:

```text
tenantId + endpoint + idempotencyKey
```

Assim, a mesma sequência usada por dois clientes diferentes não entra em conflito indevidamente.

Em sistemas com usuários, inclua no escopo uma identidade confiável obtida da autenticação. Não use somente um `userId` enviado no corpo.

## Armazenamento em PostgreSQL

Uma tabela de idempotência pode guardar o estado e a resposta original:

```sql
CREATE TABLE api_idempotency_keys (
    scope TEXT NOT NULL,
    idempotency_key TEXT NOT NULL,
    request_hash TEXT NOT NULL,
    status TEXT NOT NULL,
    response_status INTEGER,
    response_body JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at TIMESTAMPTZ,
    PRIMARY KEY (scope, idempotency_key)
);
```

A chave primária impede duas reservas iguais para o mesmo escopo. O status pode ser, por exemplo, `PROCESSING`, `SUCCEEDED` ou `FAILED`.

Uma transação pode reservar a chave e gravar o efeito de negócio de forma coordenada:

```text
iniciar transação
    inserir chave com PROCESSING
    verificar se a inserção foi a primeira
    executar alteração do negócio
    guardar resposta
    atualizar chave para SUCCEEDED ou FAILED
confirmar transação
```

Se a alteração e o registro da chave precisam ser atômicos, mantenha-os na mesma transação quando o desenho do sistema permitir. Considere locks e duração da transação para não segurar recursos por muito tempo.

## Armazenamento em Redis

[[Redis]] pode ser usado para uma reserva rápida e temporária. Uma operação equivalente precisa ser atômica, como `SET` com a opção `NX` e uma expiração:

```text
SET idem:cliente-42:chave-K PROCESSING NX EX 86400
```

`NX` significa criar somente se a chave não existir. `EX 86400` evita que um registro abandonado fique preso para sempre.

Redis pode funcionar bem como deduplicação ou lock temporário, mas é preciso decidir onde guardar o resultado completo. Se a resposta não estiver disponível depois de um restart ou expiração, o servidor deve ter uma política clara de consulta, reprocessamento ou erro.

Não use um lock temporário como se ele fosse, sozinho, uma garantia de que a operação de negócio foi concluída uma única vez.

## Qual resultado guardar?

Para devolver uma repetição com o mesmo comportamento, o servidor pode guardar:

- status HTTP;
- corpo da resposta;
- headers necessários;
- identificador do recurso criado;
- hash da requisição;
- timestamps;
- status da operação.

O corpo pode conter dados pessoais ou informações sensíveis. Proteja o armazenamento e defina uma política de retenção.

Outra opção é guardar somente o identificador do recurso e reconstruir a resposta. Essa opção reduz armazenamento, mas a resposta reconstruída pode ser diferente da original se os dados mudarem.

## Expiração da chave

As chaves não precisam ser guardadas para sempre. Defina um **TTL** (*time to live*) de acordo com o período em que o cliente pode repetir uma operação.

Considere:

- duração normal dos retries;
- atrasos de processamento;
- reconciliação financeira;
- webhooks que podem ser reenviados dias depois;
- exigências de auditoria;
- custo de armazenamento.

Apagar cedo demais pode permitir duplicidade. Guardar indefinidamente pode aumentar custo, volume e exposição de dados.

## Falhas e retries

O tratamento de falhas precisa ser definido por tipo:

- **timeout:** pode ser seguro repetir com a mesma chave;
- **erro de conexão antes do servidor processar:** repetir com a mesma chave;
- **erro temporário do provedor:** repetir com a mesma chave, com limite e espera;
- **erro de validação:** corrigir os dados ou criar outra operação;
- **erro definitivo de negócio:** normalmente devolver o mesmo erro para a mesma chave;
- **processamento assíncrono:** devolver um identificador de acompanhamento.

Não faça retry infinito. Use backoff, limite de tentativas e observabilidade.

## Webhooks e mensageria

Provedores podem enviar o mesmo webhook mais de uma vez. O evento deve ter um identificador próprio, que pode ser registrado como uma chave de deduplicação.

Em uma arquitetura com [[RabbitMQ]], [[Filas]] ou [[Broker de fila]], o broker pode entregar uma mensagem novamente quando não recebe confirmação. O consumidor deve guardar o `eventId` ou outro identificador estável e verificar se já processou aquele evento.

```text
mensagem event-123 -> processa e registra event-123
mensagem event-123 -> já registrada -> não repete o efeito
```

A chave de idempotência HTTP e o identificador de mensagem não são exatamente a mesma coisa, mas resolvem problemas relacionados de repetição.

## Idempotência e Outbox Pattern

O [[Outbox Pattern]] ajuda quando uma aplicação precisa salvar uma alteração no banco e publicar um evento.

Uma combinação comum é:

```text
requisição com chave
    ↓
transação: efeito de negócio + outbox + registro de idempotência
    ↓
publicador envia evento
    ↓
consumidor usa eventId para evitar duplicidade
```

Isso não significa que todo sistema terá exatamente uma execução física. O objetivo é que as repetições não produzam efeitos extras e que o processo possa ser retomado.

## Exemplo em Java e Quarkus

Em um recurso Java, a chave pode ser recebida como header:

```java
@POST
@Path("/pagamentos")
public Response criarPagamento(
        @HeaderParam("Idempotency-Key") String chave,
        CriarPagamentoRequest request) {

    PagamentoResponse response = service.criar(chave, request);
    return Response.status(Response.Status.CREATED)
        .entity(response)
        .build();
}
```

O `service` deve fazer a validação e a reserva de forma atômica. O recurso não deve apenas consultar a tabela e depois executar a operação fora de uma proteção contra concorrência.

Uma aplicação [[Quarkus]] pode usar CDI para injetar o serviço e um repositório. O repositório pode usar [[PostgreSQL]] diretamente ou por meio de um [[ORM]], conforme o desenho do sistema.

## Segurança

Uma chave de idempotência ajuda a controlar duplicidade, mas não autentica ninguém e não autoriza uma operação.

Boas práticas de segurança:

- exija autenticação antes de processar a operação;
- associe a chave ao usuário, conta ou tenant autenticado;
- limite tamanho e caracteres aceitos;
- rejeite chaves vazias, previsíveis ou reutilizadas fora do escopo;
- valide o fingerprint para impedir troca de payload;
- não coloque dados pessoais ou segredos dentro da chave;
- não use a chave como token de acesso;
- proteja o corpo e os headers armazenados;
- não registre chaves e respostas sensíveis em logs sem necessidade;
- aplique rate limit;
- evite devolver a resposta de uma chave para outro usuário;
- defina expiração e limpeza;
- audite operações financeiras e administrativas.

Essas medidas complementam as práticas de [[Segurança]] da API.

## Testes

Inclua casos como:

1. primeira chamada com chave válida;
2. repetição depois de sucesso;
3. repetição com corpo diferente;
4. duas chamadas simultâneas com a mesma chave;
5. chave ausente ou grande demais;
6. timeout após o efeito ter sido criado;
7. falha antes e depois da reserva;
8. expiração do registro;
9. duas contas usando o mesmo valor de chave;
10. retry de mensagem ou webhook;
11. resposta armazenada com dados sensíveis;
12. reprocessamento depois de reiniciar o serviço.

Use [[Testes]] de integração para verificar a restrição única, a transação e o comportamento real do armazenamento. Um teste unitário sozinho pode não revelar uma corrida entre duas requisições.

## Boas práticas

1. **Use uma chave por intenção de negócio.** Não use a mesma chave para pagamentos diferentes.
2. **Reutilize a chave nos retries.** Trocar a chave transforma o retry em uma nova operação.
3. **Defina o escopo.** Inclua endpoint e identidade confiável quando necessário.
4. **Faça a reserva de forma atômica.** Use restrição única, transação ou comando atômico.
5. **Compare o payload.** A mesma chave não deve aceitar dados incompatíveis.
6. **Guarde uma resposta ou uma referência estável.** Assim a repetição tem comportamento previsível.
7. **Defina estados e política para falhas.** Não deixe chaves eternamente em `PROCESSING`.
8. **Use expiração adequada.** O prazo depende da operação e dos retries esperados.
9. **Não confunda deduplicação com autorização.** A chave não substitui autenticação.
10. **Pense em concorrência.** Duas chamadas podem chegar no mesmo milissegundo.
11. **Aplique em consumidores de mensagens.** Use `eventId` ou um identificador equivalente.
12. **Monitore conflitos e reprocessamentos.** Eles mostram falhas de rede, clientes instáveis ou ataques.
13. **Documente o contrato.** Explique header, formato, duração, respostas e comportamento de erro.
14. **Teste o caminho de recuperação.** O servidor precisa saber o que fazer depois de uma queda.

## Resumo

Chave de idempotência é um identificador usado para reconhecer retries da mesma operação. O cliente reutiliza a chave; o servidor reserva, processa e guarda o resultado. A implementação precisa tratar concorrência, payload diferente, falhas, expiração, segurança e respostas repetidas. Em APIs, pagamentos, webhooks e consumidores de mensagens, ela ajuda a evitar que uma mesma intenção produza efeitos duplicados.

### Veja também

- [[Idempotência]]
- [[Requisição]]
- [[Backend]]
- [[PostgreSQL]]
- [[Redis]]
- [[RabbitMQ]]
- [[Filas]]
- [[Broker de fila]]
- [[Outbox Pattern]]
- [[cURL]]
- [[Java]]
- [[Quarkus]]
- [[ORM]]
- [[JSON]]
- [[Segurança]]
- [[Testes]]
