# PKCE

**PKCE** significa *Proof Key for Code Exchange* (prova de chave para troca de código). É uma extensão de [[Segurança]] do fluxo **Authorization Code** do [[OAuth 2.0]].

Seu objetivo é impedir que outra aplicação use um código de autorização que tenha sido interceptado antes da troca por tokens. Ele é especialmente importante para aplicações públicas (*public clients*), como aplicações mobile, desktop e frontends executados no navegador, que não conseguem proteger um `client_secret` com segurança.

## Uma analogia

Imagine que uma pessoa retire uma senha temporária em um guichê. A senha permite buscar um ingresso, mas o guichê também exige uma palavra secreta que só a pessoa original conhece.

No PKCE:

- o código de autorização é a senha temporária;
- o `code_verifier` é a palavra secreta;
- o `code_challenge` é uma versão transformada dessa palavra, apresentada antes;
- o token é o ingresso recebido depois da conferência.

Mesmo que alguém roube o código de autorização, não conseguirá trocá-lo por tokens sem o `code_verifier`.

## O problema que o PKCE resolve

No fluxo Authorization Code, a aplicação recebe um código temporário depois que a pessoa faz login e autoriza o acesso. Esse código é enviado ao servidor de autorização para ser trocado por tokens.

Se um invasor (*attacker*) conseguir interceptar o código durante o redirecionamento, ele pode tentar fazer a troca no lugar da aplicação legítima. O risco é maior em aplicações públicas, pois um segredo fixo incluído no aplicativo ou no navegador pode ser descoberto.

O PKCE associa o código a uma execução específica da aplicação. A aplicação cria um segredo temporário, e o servidor só libera os tokens quando esse segredo é apresentado corretamente.

## Termos principais

- **`code_verifier`**: valor aleatório, longo e temporário criado pela aplicação. Ele deve ser guardado até a troca do código.
- **`code_challenge`**: valor derivado do `code_verifier` e enviado no início do fluxo.
- **`S256`**: método recomendado para criar o desafio. Ele calcula [[SHA-256]] e codifica o resultado em *Base64 URL-safe*.
- **Authorization code**: código de autorização de uso único e curta duração.
- **Token endpoint**: endpoint que recebe o código e o `code_verifier` para devolver os tokens.

O servidor não precisa receber o `code_verifier` no primeiro pedido. Ele guarda o `code_challenge` e compara os valores durante a troca.

## Como o fluxo funciona

1. A aplicação gera um `code_verifier` criptograficamente aleatório.
2. A aplicação calcula o `code_challenge` usando `S256`:

   ```text
   code_challenge = BASE64URL(SHA256(code_verifier))
   ```

3. A aplicação envia o `code_challenge` ao endpoint de autorização.
4. A pessoa faz login e autoriza os escopos solicitados.
5. O servidor redireciona a aplicação para o `redirect_uri` com um código temporário.
6. A aplicação envia o código e o `code_verifier` ao endpoint de token.
7. O servidor calcula novamente o desafio e compara o resultado com o valor guardado.
8. Se a comparação for válida, o servidor troca o código por tokens.
9. A aplicação descarta o `code_verifier` depois da troca.

O invasor pode até obter o código e o `code_challenge`, mas não consegue descobrir o `code_verifier` original apenas observando o desafio gerado por `SHA-256`.

## Exemplo do pedido de autorização

O pedido inicial pode ser parecido com este:

```http
GET /authorize?
  response_type=code&
  client_id=web-app&
  redirect_uri=https%3A%2F%2Fapp.exemplo.com%2Fcallback&
  scope=openid%20profile&
  state=valor-aleatorio-da-sessao&
  code_challenge=desafio-gerado-pela-aplicacao&
  code_challenge_method=S256
```

Os espaços e quebras de linha servem apenas para facilitar a leitura. Em uma URL real, os parâmetros ficam na mesma linha e devem ser codificados corretamente.

- `response_type=code` pede um código temporário, e não tokens diretamente na URL.
- `redirect_uri` indica para onde o servidor deve voltar.
- `scope` indica quais permissões estão sendo solicitadas.
- `state` ajuda a proteger o fluxo contra solicitações forjadas (*CSRF*).
- `code_challenge` e `code_challenge_method` ativam o PKCE.

## Exemplo da troca pelo token

Depois de receber o código, a aplicação faz um pedido ao token endpoint:

```http
POST /oauth/token HTTP/1.1
Host: auth.exemplo.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
client_id=web-app&
code=codigo-temporario&
redirect_uri=https%3A%2F%2Fapp.exemplo.com%2Fcallback&
code_verifier=segredo-temporario-original
```

O servidor verifica se o `code_verifier` produz exatamente o `code_challenge` enviado no primeiro pedido. O `redirect_uri` também deve ser igual ao usado no início do fluxo.

## Exemplo conceitual em [[JavaScript]]

Uma aplicação pode seguir esta ideia usando recursos criptográficos do navegador:

```javascript
const codeVerifier = gerarValorAleatorioSeguro();
const codeChallenge = base64Url(await sha256(codeVerifier));

sessionStorage.setItem("pkce_code_verifier", codeVerifier);

// Envie codeChallenge e codeChallengeMethod=S256 para /authorize.
// Depois do redirecionamento, recupere o verifier e envie-o para /oauth/token.
const verifier = sessionStorage.getItem("pkce_code_verifier");
```

As funções `gerarValorAleatorioSeguro`, `sha256` e `base64Url` representam operações que devem usar APIs criptográficas confiáveis, como a Web Crypto API. Não se deve substituir um gerador seguro por `Math.random()`.

## Boas práticas

- Usar sempre `S256`; o método `plain` oferece proteção muito menor e normalmente deve ser recusado.
- Gerar o `code_verifier` com um gerador criptograficamente seguro, com tamanho e formato aceitos pelo provedor de identidade.
- Manter o `code_verifier` apenas durante a tentativa atual e removê-lo após a troca ou quando o fluxo falhar.
- Usar `state` para associar a resposta à sessão que iniciou o fluxo.
- Usar `nonce` também quando o fluxo envolver [[OIDC]], para associar o ID token ao pedido original.
- Usar HTTPS em todas as etapas, evitando que códigos, desafios ou tokens sejam observados no caminho.
- Cadastrar e comparar `redirect_uri` com correspondência exata; não aceitar redirecionamentos genéricos.
- Fazer o código de autorização expirar rapidamente e permitir seu uso apenas uma vez.
- Nunca registrar em logs o `code_verifier`, códigos de autorização, tokens ou dados sensíveis.
- Validar corretamente os tokens recebidos: assinatura, emissor (*issuer*), público (*audience*) e validade.
- Não colocar um `client_secret` no código de um frontend ou aplicativo mobile. Um segredo distribuído ao usuário não é realmente secreto.

O PKCE reduz o risco de interceptação do código, mas não substitui HTTPS, `state`, validação de tokens ou o controle correto das permissões (*scopes*).

## PKCE, OAuth 2.0 e OIDC

O [[OAuth 2.0]] define como uma aplicação pode obter autorização para acessar recursos. O [[OIDC]] adiciona identidade e login sobre o OAuth 2.0. O PKCE protege especificamente a troca do código de autorização por tokens.

Assim, eles têm papéis diferentes:

| Conceito | Responsabilidade |
| --- | --- |
| OAuth 2.0 | Autorizar acesso a recursos |
| OIDC | Informar a identidade da pessoa autenticada |
| PKCE | Proteger a troca do código de autorização |

Em aplicações atuais, é comum usar Authorization Code com PKCE junto com OIDC quando a aplicação precisa fazer login com segurança.
