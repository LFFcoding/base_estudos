# JSON Web Token (JWT)

**JWT** significa *JSON Web Token*. É um formato compacto para transportar informações (*claims*) entre partes, normalmente em uma aplicação que usa [[OAuth 2.0]] ou [[OIDC]]. Para uma visão mais ampla dos cuidados envolvidos, consulte [[Segurança]].

Um JWT é como um crachá digital: ele pode informar quem é o portador, para quem o crachá foi emitido e até quando é válido. A assinatura permite verificar se o conteúdo foi alterado e se foi produzido por uma fonte confiável.

## JWT não é uma senha nem uma sessão completa

Um JWT é apenas um formato de dados assinado. Ele não define sozinho:

- como a pessoa fará login;
- quais permissões ela terá;
- como um token será revogado imediatamente;
- onde o token será armazenado;
- como a aplicação responderá a um roubo de credencial.

Essas decisões fazem parte da arquitetura de segurança. O [[OAuth 2.0]] define fluxos de autorização, o [[OIDC]] acrescenta identidade e o [[PKCE]] protege a troca do código de autorização.

## As três partes

Um JWT normalmente tem três partes separadas por pontos:

```text
header.payload.signature
```

### Header

O **header** informa metadados sobre o token, como o tipo e o algoritmo usado na assinatura:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "chave-2026-01"
}
```

- `alg` informa o algoritmo esperado;
- `typ` indica o tipo do token;
- `kid` (*key ID*) ajuda o servidor a escolher a chave pública correta quando existem várias chaves ativas.

### Payload

O **payload** contém as informações chamadas de *claims*:

```json
{
  "iss": "https://auth.exemplo.com",
  "sub": "usuario-123",
  "aud": "api.exemplo.com",
  "exp": 1797638400,
  "iat": 1797634800,
  "scope": "pedidos:read"
}
```

Claims comuns:

- `iss` (*issuer*): quem emitiu o token;
- `sub` (*subject*): a identidade representada;
- `aud` (*audience*): para quem o token foi criado;
- `exp` (*expiration time*): quando o token expira;
- `iat` (*issued at*): quando foi emitido;
- `nbf` (*not before*): a partir de quando pode ser aceito;
- `jti` (*JWT ID*): identificador único do token;
- `scope` ou `roles`: permissões que podem ser consideradas pela API.

O payload é codificado, não necessariamente criptografado. Portanto, não coloque nele senhas, chaves privadas, dados bancários ou qualquer informação que precise permanecer secreta.

### Signature

A **signature** é calculada a partir do header, do payload e de uma chave. Ela permite que o servidor verifique se o token foi alterado:

```text
assinatura = algoritmo(header + "." + payload, chave)
```

Se alguém modificar `sub`, `aud` ou `scope`, a assinatura original deixará de corresponder ao conteúdo.

## Como um JWT é usado

Depois de receber um access token, o cliente pode enviá-lo para a API em uma [[Requisição]] HTTP:

```http
GET /api/pedidos HTTP/1.1
Host: api.exemplo.com
Authorization: Bearer <jwt>
```

O [[Backend]] deve validar o token antes de responder. Uma validação mínima costuma verificar:

1. se o token tem três partes e pode ser interpretado;
2. se o algoritmo está na lista permitida pelo servidor;
3. se a assinatura é válida usando uma chave confiável;
4. se `iss` é o emissor esperado;
5. se `aud` corresponde à API;
6. se `exp` ainda não passou e `nbf`, quando presente, já foi atingido;
7. se o escopo ou a role permite a operação solicitada;
8. se o recurso acessado pertence à pessoa ou ao serviço representado por `sub`.

Decodificar o JWT e ler seu payload não é validá-lo. O frontend nunca deve ser a única parte responsável por essa verificação.

## Algoritmos e chaves

Os algoritmos mais comuns são:

- **HS256**: usa a mesma chave secreta para assinar e validar. É simples, mas todos os serviços que validam precisam conhecer o mesmo segredo.
- **RS256**: usa uma chave privada para assinar e uma chave pública para validar. Isso facilita distribuir apenas a parte pública às APIs.
- **ES256**: usa criptografia de curva elíptica e também trabalha com par de chaves pública e privada.

Em sistemas com vários serviços, RS256 ou ES256 costumam ser convenientes porque o serviço emissor mantém a chave privada e os validadores recebem somente a chave pública.

Nunca escolha o algoritmo apenas porque ele apareceu no header recebido. A aplicação deve configurar quais algoritmos aceita e rejeitar opções inesperadas, incluindo `none`.

## JWT e refresh token

Um access token JWT geralmente tem vida curta. Quando ele expira, o cliente pode usar um refresh token para pedir outro, conforme o fluxo configurado no [[OAuth 2.0]].

Como um JWT válido normalmente continua válido até `exp`, revogá-lo imediatamente pode ser difícil. Por isso, use expiração curta, rotação de refresh tokens, revogação quando necessário e uma estratégia para bloquear sessões comprometidas.

## JWT no OIDC

No [[OIDC]], o **ID token** normalmente é um JWT. Ele informa à aplicação quem foi autenticado e contém claims como `iss`, `sub`, `aud` e `exp`.

O access token tem outra finalidade: permitir que uma API aceite uma operação autorizada. Mesmo quando os dois são JWTs, eles não são automaticamente intercambiáveis. A API deve aceitar somente tokens cujo `aud` e demais características indiquem que foram criados para ela.

## Boas práticas

- use bibliotecas mantidas e específicas para JWT; não implemente a criptografia manualmente;
- valide assinatura, emissor, audiência, validade e permissões no servidor;
- configure uma lista explícita de algoritmos permitidos;
- mantenha a chave privada protegida e faça rotação planejada;
- use `kid` para permitir a transição entre chaves, sem quebrar tokens ainda válidos;
- mantenha access tokens curtos e com o menor conjunto de claims necessário;
- não coloque dados secretos ou informações que não deveriam ser lidas no payload;
- não coloque tokens em URLs, logs, mensagens de erro ou repositórios;
- use HTTPS em todas as comunicações;
- trate armazenamento no navegador com cuidado e reduza a exposição a XSS;
- não confie em `role`, `scope` ou `sub` enviados separadamente pelo cliente;
- registre eventos úteis para investigação, mas remova ou masque o token.

JWT pode ser uma boa solução para APIs distribuídas, mas não é obrigatório. Em alguns sistemas, uma sessão opaca armazenada no servidor facilita revogação e controle centralizado. A escolha depende dos riscos, da arquitetura e da necessidade de compartilhar a validação entre serviços.
