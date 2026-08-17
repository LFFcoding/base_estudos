# Requisição

Em desenvolvimento de sistemas, uma **requisição** (*request*) é uma mensagem enviada por um programa para pedir alguma coisa a outro programa.

Quando o [[Frontend]] precisa buscar ou enviar informações, ele faz uma requisição para o [[Backend]]. O backend recebe essa mensagem, processa o pedido e devolve uma resposta (*response*).

Uma analogia é pedir comida em um restaurante:

- você é o cliente;
- o cardápio e o garçom representam a [[API]];
- o pedido que você faz é a requisição;
- a cozinha representa o backend;
- o prato pronto é a resposta.

## Requisições HTTP

Na web, as requisições normalmente usam o protocolo HTTP (*Hypertext Transfer Protocol*). Uma requisição HTTP é como um envelope: ela leva instruções, informações sobre o pedido e, quando necessário, os dados enviados.

As partes mais comuns são:

1. linha inicial (*request line*);
2. URL e parâmetros;
3. cabeçalhos (*headers*);
4. corpo (*body*);
5. cookies (*cookies*), que são enviados dentro de um header.

Nem toda requisição possui todas essas partes. Uma requisição `GET` simples pode ter apenas a linha inicial e alguns headers, sem body.

## 1. Linha inicial

A linha inicial informa o método, o recurso desejado e a versão do HTTP:

```http
GET /usuarios HTTP/1.1
```

Ela possui:

- **método HTTP:** indica a ação desejada;
- **recurso ou caminho:** indica o que será acessado;
- **versão do protocolo:** indica a versão do HTTP usada na comunicação.

Os métodos mais comuns são:

- `GET`: buscar informações;
- `POST`: criar algo ou enviar dados para um processamento;
- `PUT`: substituir um recurso inteiro;
- `PATCH`: alterar parte de um recurso;
- `DELETE`: excluir um recurso.

O método ajuda a deixar clara a intenção do pedido. Uma requisição `GET`, por exemplo, deve buscar informações e não apagar dados.

## 2. URL e parâmetros

A URL (*Uniform Resource Locator*) indica o endereço do recurso. Ela pode conter diferentes tipos de informação.

### Caminho (*path*)

Indica o recurso ou o identificador do recurso:

```http
/usuarios/42
```

Nesse exemplo, `42` pode ser o identificador do usuário que será consultado.

### Parâmetros de consulta (*query parameters*)

São colocados depois de `?` e servem para filtrar, ordenar ou paginar resultados:

```http
/usuarios?pagina=2&limite=20
```

Nesse caso, `pagina` e `limite` são parâmetros de consulta.

### Fragmento (*fragment*)

É a parte depois de `#`, como `#detalhes`. O navegador usa essa informação para navegar dentro da página, mas normalmente não a envia ao servidor na requisição HTTP.

## 3. Cabeçalhos (*headers*)

Headers são pares de nome e valor que carregam metadados, ou seja, informações sobre a requisição. Eles não costumam ser o conteúdo principal do pedido; servem para explicar como o pedido deve ser interpretado.

Exemplos importantes:

- `Host`: informa o domínio que deve receber a requisição;
- `Content-Type`: informa o formato do body, como `application/json`;
- `Accept`: informa quais formatos de resposta o cliente aceita;
- `Authorization`: envia dados usados para autenticação, como um token;
- `Cookie`: envia cookies associados ao domínio;
- `User-Agent`: identifica o programa ou navegador que fez a requisição;
- `Cache-Control`: orienta regras de armazenamento temporário (*cache*);
- `X-Request-ID`: pode carregar um identificador para acompanhar o pedido nos logs.

Um header não deve ser aceito cegamente como prova de identidade ou permissão. O [[Backend]] precisa validar a autenticação e a autorização no servidor.

## 4. Corpo (*body*)

O body contém os dados enviados no pedido. Ele é mais comum em requisições `POST`, `PUT` e `PATCH`, mas seu uso depende do contrato da [[API]].

O formato do body é informado pelo header `Content-Type`. Alguns formatos comuns são:

- `application/json`: usado para enviar objetos e listas em [[JSON]];
- `application/x-www-form-urlencoded`: usado em formulários simples;
- `multipart/form-data`: usado para enviar arquivos junto com campos;
- `text/plain`: usado para enviar texto simples.

Exemplo de body em [[JSON]]:

```json
{
  "nome": "Ana",
  "email": "ana@example.com"
}
```

O body pode ser vazio. Por exemplo, um `DELETE` pode identificar o recurso somente pelo caminho da URL.

## 5. Cookies (*cookies*)

Cookies são pequenos valores que o navegador guarda e envia novamente em requisições futuras para o mesmo domínio. Eles podem ser usados para manter uma sessão de login, salvar preferências ou ajudar a medir o uso da aplicação.

Apesar de aparecerem como a parte `Cookie` de um header, eles merecem atenção especial porque podem carregar informações de sessão. Cookies de autenticação devem usar, quando possível:

- `Secure`, para serem enviados apenas por HTTPS;
- `HttpOnly`, para impedir acesso por JavaScript no navegador;
- `SameSite`, para reduzir o risco de ataques CSRF (*Cross-Site Request Forgery*).

## Exemplo completo

Ao cadastrar uma pessoa, o frontend pode enviar:

```http
POST /usuarios?origem=web HTTP/1.1
Host: api.exemplo.com
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
X-Request-ID: 7f3a1c

{
  "nome": "Ana",
  "email": "ana@example.com"
}
```

Nesse exemplo:

- `POST` informa que o pedido pretende criar ou processar algo;
- `/usuarios` é o caminho do recurso;
- `origem=web` é um parâmetro de consulta;
- `Host`, `Content-Type`, `Accept`, `Authorization` e `X-Request-ID` são headers;
- o JSON é o body;
- `<token>` é apenas um exemplo e não deve ser um segredo real em documentação ou código público.

Essa requisição diz ao backend: “crie um usuário usando estes dados”. O backend valida as informações, verifica a autorização, salva o cadastro e devolve uma resposta informando o resultado.

## O que não faz parte da requisição?

O **código de status** (`200`, `201`, `400`, `404`, `500`) pertence à resposta, não à requisição. Ele informa como o servidor tratou o pedido.

Da mesma forma, o resultado devolvido pelo servidor, como um usuário criado ou uma mensagem de erro, faz parte da resposta (*response*).

## Caminho de uma requisição

1. a pessoa realiza uma ação na tela;
2. o [[Frontend]] monta a requisição;
3. a [[API]] encaminha a requisição para o [[Backend]];
4. o backend valida os dados e aplica as regras de negócio;
5. o backend consulta ou altera informações, quando necessário;
6. o backend devolve uma resposta para o frontend;
7. o frontend mostra o resultado na tela.

Esse caminho faz parte da comunicação entre as camadas da [[Stack]] da aplicação.

## Boas práticas

- escolher o método HTTP adequado, deixando clara a intenção da requisição;
- usar URLs previsíveis e nomes consistentes para os recursos;
- enviar apenas os dados necessários, reduzindo exposição e desperdício;
- definir um formato de dados claro, como JSON, e informá-lo no `Content-Type`;
- usar códigos de status e mensagens de erro consistentes na resposta;
- definir limites de tamanho para URLs, headers e body;
- configurar tempo limite (*timeout*) e tratar falhas de rede;
- usar um identificador de requisição (*request ID*) para acompanhar o pedido nos logs;
- tornar operações que podem ser repetidas seguras contra duplicação, usando idempotência (*idempotency*) quando necessário.

## Segurança

Uma requisição pode carregar dados pessoais, credenciais e comandos importantes. Por isso, o [[Backend]] nunca deve confiar cegamente no que vem do cliente.

### Proteja os dados no caminho

Use HTTPS, que combina HTTP com TLS (*Transport Layer Security*), para criptografar os dados entre o cliente e o servidor. Sem HTTPS, alguém na rede pode tentar observar ou alterar a comunicação.

### Valide tudo no servidor

O frontend pode ajudar o usuário com validações rápidas, mas a validação que realmente protege o sistema deve acontecer no backend. Confira formato, tamanho, campos obrigatórios e valores permitidos.

### Controle identidade e permissão

Autenticação (*authentication*) responde “quem é a pessoa?”. Autorização (*authorization*) responde “o que essa pessoa pode fazer?”. As duas verificações devem ocorrer no servidor, em cada rota protegida.

### Não exponha segredos

Não coloque senhas, tokens ou chaves privadas em URLs, porque URLs podem aparecer no histórico do navegador, em logs e em ferramentas de monitoramento. Também não registre esses valores em logs. Use placeholders como `<token>` em exemplos.

### Proteja contra abuso

Use limite de requisições (*rate limiting*), tamanho máximo de body, tempo limite e bloqueios adequados. Essas medidas ajudam contra excesso de tráfego, tentativas repetidas de login e requisições criadas para consumir muitos recursos.

### Proteja sessões e formulários

Ao usar autenticação por cookies, configure `Secure`, `HttpOnly` e `SameSite` e avalie proteção contra CSRF. Também configure o CORS (*Cross-Origin Resource Sharing*) para permitir somente origens realmente necessárias.

### Evite comandos perigosos

Nunca monte consultas SQL, comandos do sistema ou HTML juntando diretamente valores recebidos na requisição. Use parâmetros, validação e técnicas próprias de proteção contra injeção (*injection*).

## Resumo

> **Requisição é uma mensagem enviada a um sistema para pedir uma ação ou informação.**

Ela pode carregar uma instrução na linha inicial, parâmetros na URL, metadados nos headers, dados no body e informações de sessão em cookies. A resposta é o resultado desse pedido.
