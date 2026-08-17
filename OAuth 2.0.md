# OAuth 2.0

**OAuth 2.0** é um framework de autorização (*authorization framework*). Ele permite que uma aplicação acesse um recurso em nome de uma pessoa ou de outro sistema, sem precisar receber a senha desse usuário.

Em palavras simples, OAuth 2.0 responde:

> “O que este acesso pode fazer?”

Ele não responde sozinho:

> “Quem é esta pessoa?”

Para identidade e login, o protocolo mais comum é o [[OIDC]] (*OpenID Connect*), que é construído sobre OAuth 2.0.

Uma analogia é um hotel:

- a pessoa é a dona dos recursos;
- a aplicação é um serviço que precisa entrar em alguns quartos;
- a recepção confirma a autorização;
- o access token é um cartão com acesso limitado;
- o cartão informa quais áreas podem ser acessadas e por quanto tempo;
- a aplicação nunca precisa receber a chave principal do hotel.

## Exemplo simples

Imagine uma aplicação de agenda que quer ler os eventos de uma pessoa. Em vez de pedir a senha do calendário, ela redireciona a pessoa para o provedor de autorização.

A pessoa faz login no provedor, escolhe o que deseja compartilhar e a aplicação recebe um token com permissões limitadas. Depois, usa esse token para chamar a API do calendário.

## Papéis do OAuth 2.0

OAuth 2.0 define papéis (*roles*) diferentes:

- **Resource Owner:** pessoa ou sistema que possui o recurso;
- **Client:** aplicação que quer acessar o recurso;
- **Authorization Server:** servidor que autentica e emite tokens;
- **Resource Server:** API que protege os recursos e valida os tokens.

Uma mesma empresa pode ter o servidor de autorização e o servidor de recursos, mas as responsabilidades continuam diferentes.

## Tokens

### Access token

O **access token** é apresentado à API para provar que a aplicação recebeu determinada autorização:

```http
GET /agenda/eventos
Authorization: Bearer <access-token>
```

O token deve ter duração limitada e escopos (*scopes*) que indiquem o que ele pode fazer, como `agenda:read` ou `agenda:write`.

Um access token pode ser um [[JWT]], mas isso não é obrigatório. OAuth 2.0 define a autorização e os fluxos; o formato do token pode variar conforme o servidor.

### Refresh token

O **refresh token** pode ser usado para pedir um novo access token quando o atual expirar, sem exigir que a pessoa faça login novamente.

Ele normalmente é mais sensível que o access token e deve ser armazenado com proteção maior. Nem todo fluxo precisa usar refresh token.

### Authorization code

O **authorization code** é um código temporário devolvido depois da autorização. A aplicação troca esse código por tokens através de uma comunicação com o servidor de autorização.

O código não deve ser tratado como access token e não deve ser reutilizado.

## Authorization Code com PKCE

O fluxo recomendado para aplicações web, mobile e outras aplicações públicas é o **Authorization Code Flow com [[PKCE]]** (*Proof Key for Code Exchange*).

O caminho simplificado é:

1. a aplicação cria um `code_verifier` aleatório;
2. a aplicação cria um `code_challenge` a partir desse valor;
3. a aplicação redireciona a pessoa ao servidor de autorização;
4. a pessoa faz login e autoriza os escopos pedidos;
5. o servidor redireciona de volta com um código temporário;
6. a aplicação envia o código e o `code_verifier` ao endpoint de token;
7. o servidor verifica o PKCE e devolve os tokens.

O `code_verifier` funciona como uma prova de que a aplicação que iniciou o pedido é a mesma que está trocando o código. Ele dificulta o uso de um código interceptado por outra aplicação.

## Client Credentials

O fluxo **Client Credentials** é usado quando um sistema acessa outro sistema sem uma pessoa participando do login.

Por exemplo, um pipeline de [[CI-CD]] pode obter autorização para publicar uma imagem ou um serviço pode chamar uma API interna.

Um pedido de token pode ter este formato ilustrativo:

```http
POST /oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&scope=deploy:write
```

O cliente deve se autenticar de forma segura. Não coloque `client_secret` em código frontend, aplicativos públicos ou repositórios.

## Outros fluxos

- **Refresh Token:** obtém um novo access token sem repetir todo o login;
- **Device Authorization:** usado em TVs, consoles e dispositivos com entrada limitada;
- **Authorization Code com PKCE:** indicado para aplicações públicas e interativas.

O antigo fluxo Implicit e o fluxo que enviava diretamente a senha do usuário para o cliente não devem ser escolhidos em aplicações novas. Eles aumentam riscos e têm alternativas mais seguras.

## OAuth 2.0 e OIDC

- OAuth 2.0 delega acesso a recursos e APIs;
- [[OIDC]] adiciona identidade e login sobre OAuth 2.0;
- access token é destinado ao servidor de recursos;
- ID token, normalmente um [[JWT]], é destinado à aplicação cliente para informar a identidade autenticada.

Não use um ID token automaticamente para chamar uma API. A API deve receber e validar um access token criado para ela.

## Como a API valida o token?

O [[Backend]] deve verificar, conforme o formato e o provedor:

- assinatura e chave do token;
- emissor (*issuer*);
- audiência (*audience*);
- expiração;
- escopos e permissões;
- cliente e contexto esperados.

Receber um token não significa que a ação está permitida. O backend ainda deve verificar se o escopo permite aquela operação e se a pessoa ou serviço tem acesso ao recurso específico.

## Boas práticas e [[Segurança]]

- usar HTTPS em todas as etapas;
- usar Authorization Code com [[PKCE]] em aplicações públicas;
- pedir somente os escopos necessários, seguindo o menor privilégio;
- usar URLs de redirecionamento exatas e previamente registradas;
- gerar `state` imprevisível para proteger o fluxo contra falsificação;
- usar `nonce` quando o fluxo também envolver [[OIDC]];
- manter access tokens curtos e renovar de forma controlada;
- proteger e, quando possível, rotacionar refresh tokens;
- guardar tokens em local seguro, nunca em URLs ou logs;
- manter client secrets somente em aplicações capazes de protegê-los, como o backend;
- validar tokens no servidor de recursos, não apenas no frontend;
- não aceitar qualquer issuer, audience ou algoritmo informado pelo cliente;
- revogar credenciais quando houver suspeita de exposição;
- registrar eventos de segurança sem registrar o conteúdo dos tokens;
- revisar regularmente clientes, escopos e permissões existentes.

## Resumo

> **OAuth 2.0 é um framework que permite conceder acesso limitado a recursos sem compartilhar a senha do usuário com a aplicação.**

Ele usa tokens, escopos e fluxos de autorização. Para representar a identidade de quem fez login, use OIDC sobre OAuth 2.0.
