# OpenID Connect (OIDC)

**OIDC** significa *OpenID Connect*. É um protocolo de autenticação construído sobre o [[OAuth 2.0]]. Ele permite que uma aplicação descubra e confirme a identidade de uma pessoa ou de outro sistema.

Em palavras simples, OIDC responde:

> “Quem está fazendo este acesso?”

O OAuth 2.0 responde principalmente:

> “O que este acesso pode fazer?”

Uma analogia é uma portaria:

- o provedor de identidade é a administração do prédio;
- a autenticação confirma quem é a pessoa;
- o **ID token** é um documento que informa a identidade confirmada;
- o **access token** é um cartão com permissões para acessar determinados espaços;
- a aplicação verifica os documentos antes de permitir o acesso.

## O que o OIDC acrescenta ao OAuth?

O OAuth 2.0 foi criado principalmente para delegar acesso a recursos. O OIDC adiciona uma forma padronizada de autenticar e obter informações sobre a identidade.

O OIDC normalmente usa:

- **ID token:** token com informações sobre a identidade autenticada;
- **Access token:** credencial usada para acessar uma API autorizada;
- **UserInfo endpoint:** endereço que pode fornecer informações do usuário;
- **Discovery document:** documento que informa os endpoints, chaves e configurações do provedor;
- **Provider:** serviço que autentica e emite tokens, como um provedor de identidade.

O ID token não deve ser usado automaticamente como access token. Cada token possui uma finalidade diferente.

## Fluxo de login

Um fluxo comum para uma aplicação web é:

1. a pessoa tenta entrar na aplicação;
2. a aplicação redireciona para o provedor OIDC;
3. a pessoa informa suas credenciais no provedor;
4. o provedor autentica a pessoa e pede consentimento, quando necessário;
5. o provedor devolve um código temporário para a aplicação;
6. a aplicação troca o código por tokens;
7. a aplicação valida o ID token;
8. a aplicação cria uma sessão e usa o access token quando precisa chamar uma API.

Esse fluxo é conhecido como **Authorization Code Flow**. Para aplicações públicas, como as executadas no navegador, use PKCE (*Proof Key for Code Exchange*) para dificultar o uso indevido de um código interceptado.

## O que deve ser validado?

Uma aplicação não deve aceitar um token apenas porque ele parece um JSON válido. Ela precisa validar, entre outras coisas:

- assinatura do token;
- emissor (*issuer*), confirmando quem o emitiu;
- audiência (*audience*), confirmando para qual aplicação o token foi criado;
- validade e tempo de expiração;
- nonce, quando usado no fluxo de login;
- escopos e permissões necessários;
- estado (*state*) da requisição, reduzindo riscos de falsificação.

O [[Backend]] deve verificar as permissões antes de executar uma ação. Saber quem é a pessoa não significa que ela pode acessar todos os recursos.

## OIDC para usuários e para sistemas

OIDC pode ser usado em dois cenários diferentes:

- **login de usuário:** uma pessoa entra no [[Frontend]] por meio de um provedor de identidade;
- **identidade de workload:** um pipeline ou serviço prova sua identidade para outro serviço sem usar uma senha permanente.

No segundo caso, OIDC é especialmente útil em [[CI-CD]] e [[DevOps]].

## OIDC no GitHub Actions e AWS

Um workflow do [[GitHub]] pode usar OIDC para obter credenciais temporárias da AWS:

1. um job do GitHub Actions solicita um token OIDC;
2. o token informa qual repositório, branch e workflow estão executando;
3. o AWS IAM verifica se confia no provedor e nas condições do token;
4. o job assume uma role autorizada;
5. a AWS fornece credenciais temporárias;
6. o job publica uma imagem no ECR ou faz um deploy.

Essa abordagem evita guardar uma chave de acesso permanente da AWS como secret do GitHub. O OIDC não elimina a necessidade de permissões corretas: a role ainda deve ter somente os acessos necessários.

Uma política de confiança simplificada pode conter condições como:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    },
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:exemplo/projeto:ref:refs/heads/main"
    }
  }
}
```

Essa política é apenas um exemplo. Em um ambiente real, substitua os valores, limite o repositório e a branch e conceda à role somente as permissões necessárias.

## OIDC, OAuth e SAML

- **OIDC:** autenticação e identidade sobre OAuth 2.0;
- **OAuth 2.0:** delegação de acesso a recursos, como uma API;
- **SAML:** padrão usado com frequência em autenticação empresarial e login único (*Single Sign-On* ou SSO).

Eles podem participar de arquiteturas parecidas, mas não são nomes diferentes para o mesmo protocolo.

## Boas práticas e segurança

- usar Authorization Code Flow com PKCE em aplicações públicas;
- validar assinatura, emissor, audiência, expiração e escopos dos tokens;
- não aceitar qualquer issuer ou audience informado pelo cliente;
- armazenar tokens com cuidado e limitar sua duração;
- não colocar tokens em URLs, pois URLs podem aparecer em histórico e logs;
- usar HTTPS em todas as etapas;
- pedir somente os escopos necessários;
- separar roles e permissões por ambiente;
- restringir a confiança OIDC a repositórios, branches e ambientes específicos;
- preferir credenciais temporárias a chaves permanentes;
- não registrar tokens nos logs;
- revisar alterações na política de confiança e nas permissões IAM;
- planejar revogação e resposta caso um token ou segredo seja exposto.

## Resumo

> **OpenID Connect (OIDC) é um protocolo que permite confirmar identidades usando tokens e o modelo de autorização do OAuth 2.0.**

Ele pode proteger logins de usuários e permitir que pipelines, como GitHub Actions, acessem a AWS com credenciais temporárias.
