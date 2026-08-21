# TLS

**TLS** significa *Transport Layer Security* (segurança da camada de transporte). É um protocolo usado para proteger dados enquanto eles viajam entre dois pontos, como um navegador e um servidor.

Quando uma página usa `https://`, o HTTP está sendo transportado por uma conexão protegida por TLS. O TLS também pode proteger conexões de outros serviços, como banco de dados, e-mail e cache.

## Uma analogia

Imagine enviar um documento importante por uma transportadora:

- uma embalagem lacrada impede que outras pessoas leiam o conteúdo;
- um selo mostra se alguém abriu ou alterou a embalagem;
- um documento de identidade ajuda a confirmar que a entrega chegou ao destinatário correto.

O TLS oferece proteções parecidas para uma comunicação digital:

- **confidencialidade**: dificulta que terceiros leiam os dados;
- **integridade**: ajuda a detectar alterações durante o caminho;
- **autenticação**: ajuda o cliente a confirmar a identidade do servidor.

## O que acontece sem TLS?

Em uma conexão HTTP sem proteção, alguém que consiga observar a rede pode tentar ler ou alterar:

- senhas e dados de login;
- tokens e cookies de sessão;
- informações pessoais enviadas em uma [[Requisição]];
- respostas devolvidas pelo [[Backend]].

O TLS protege o caminho entre os pontos, mas não corrige uma aplicação que possui falhas de autorização, armazena senhas de forma insegura ou entrega dados para a pessoa errada.

## Como funciona o handshake

Antes de transmitir os dados da aplicação, cliente e servidor fazem um **TLS handshake** (aperto de mãos):

1. o cliente informa versões e algoritmos que consegue usar;
2. o servidor escolhe os parâmetros da conexão e apresenta seu certificado;
3. o cliente verifica o certificado, o domínio, a validade e a cadeia de confiança;
4. cliente e servidor realizam uma troca de chaves;
5. os dois calculam chaves de sessão temporárias;
6. os dados da aplicação passam a ser protegidos com criptografia simétrica e verificação de integridade.

A criptografia assimétrica ajuda a autenticar o servidor e a estabelecer a sessão. Depois disso, a criptografia simétrica é usada para transportar muitos dados com mais eficiência.

## Certificados digitais

Um certificado TLS é um documento digital, normalmente no formato X.509, que associa um domínio a uma chave pública.

Ele pode informar:

- o domínio protegido;
- a chave pública do servidor;
- a autoridade certificadora (*Certificate Authority* ou CA);
- o período de validade;
- os nomes alternativos do domínio (*Subject Alternative Names* ou SANs);
- a assinatura da autoridade certificadora.

O servidor guarda a chave privada correspondente. Ela deve permanecer secreta: quem possui essa chave pode conseguir se passar pelo servidor em determinadas situações.

O navegador confia em autoridades certificadoras conhecidas. Ele verifica a cadeia que liga o certificado do domínio a uma autoridade raiz confiável.

## TLS, HTTPS e SSL

- **TLS** é o protocolo de segurança;
- **HTTPS** é HTTP protegido por TLS;
- **SSL** foi o predecessor do TLS e as versões antigas não devem ser usadas;
- **SSH** também cria um canal seguro, mas é outro protocolo, normalmente usado para acesso administrativo a servidores.

Uma URL como esta indica HTTPS:

```text
https://app.exemplo.com/usuarios
```

O `https` não garante que a aplicação seja segura em todos os aspectos. Ele indica que a comunicação com aquele domínio foi estabelecida usando uma camada TLS válida.

## Exemplo com Caddy

O [[Caddy]] pode receber HTTPS na frente do [[Frontend]] e do [[Backend]]:

```text
app.exemplo.com {
    reverse_proxy backend:8080
}
```

Quando o domínio real aponta para o servidor e as portas necessárias estão acessíveis, o Caddy pode obter e renovar certificados automaticamente. Ele termina a conexão TLS e encaminha a requisição para o serviço interno conforme a configuração.

Mesmo quando o proxy faz a terminação TLS, a equipe deve decidir se a comunicação entre o proxy e o backend também precisa ser criptografada. Em redes privadas e controladas o risco pode ser diferente, mas não desaparece automaticamente.

## Como verificar uma conexão

Para observar apenas os cabeçalhos de uma conexão HTTPS, use [[cURL]]:

```bash
curl -I https://app.exemplo.com
```

Para inspecionar o certificado e os parâmetros negociados, uma ferramenta como OpenSSL pode ser usada:

```bash
openssl s_client \
  -connect app.exemplo.com:443 \
  -servername app.exemplo.com
```

Esses comandos ajudam a investigar validade, domínio e cadeia do certificado. Eles não substituem uma análise completa da configuração.

## TLS em conexões internas

TLS não serve apenas para o navegador. Dependendo da arquitetura, também pode proteger:

- a conexão entre o [[Backend]] e o [[PostgreSQL]];
- a comunicação entre serviços;
- conexões com um [[Redis]];
- acesso a APIs externas;
- tráfego entre um proxy e os serviços internos.

Em uma conexão JDBC com PostgreSQL, por exemplo, a configuração pode exigir TLS:

```text
jdbc:postgresql://database:5432/app?sslmode=require
```

Em produção, avalie uma configuração que também valide a identidade do servidor, como `verify-full`, com a autoridade certificadora configurada corretamente. Criptografar o canal sem validar o destino pode deixar espaço para ataques de servidor intermediário (*man-in-the-middle*).

## TLS não substitui autenticação e autorização

TLS ajuda a proteger o caminho, mas o sistema ainda precisa decidir quem pode fazer cada ação.

Por exemplo:

1. TLS protege a senha durante o envio;
2. o servidor autentica a pessoa;
3. o servidor verifica se ela tem permissão;
4. o backend aplica as regras de negócio;
5. o banco recebe somente a operação autorizada.

Essa separação conecta TLS às práticas de [[Segurança]], mas os conceitos não são iguais. TLS protege o transporte; autenticação identifica; autorização limita ações.

## Boas práticas

- use TLS em toda aplicação acessível pela rede, especialmente em produção;
- prefira versões modernas, normalmente TLS 1.2 ou TLS 1.3 conforme a compatibilidade necessária;
- desative SSL e versões antigas de TLS;
- use certificados emitidos para os domínios corretos;
- renove certificados antes da expiração e monitore o processo;
- proteja a chave privada com permissões restritas e um gerenciador de segredos;
- redirecione HTTP para HTTPS quando fizer sentido;
- use HSTS somente depois de confirmar que todos os acessos necessários funcionam por HTTPS;
- evite conteúdo misto, como uma página HTTPS carregando scripts por HTTP;
- valide certificados em clientes e serviços internos;
- não desative a validação TLS para “resolver” erros de certificado;
- considere mTLS (*mutual TLS*) quando o servidor também precisar autenticar o cliente por certificado;
- teste renovação, expiração, cadeia de certificados e recuperação de falhas;
- não coloque senhas ou chaves privadas em exemplos, logs ou no [[GitHub]].

## TLS em uma VPS e na AWS

Em uma [[VPS]], a equipe normalmente configura o servidor web, portas, certificados, renovação, permissões da chave privada e regras de firewall. O [[Caddy]] pode automatizar parte da emissão e da renovação.

Na AWS, o [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) gerencia certificados públicos e privados. Certificados públicos do ACM podem ser associados a serviços integrados, como Application Load Balancer, CloudFront e API Gateway. Nesses casos, a AWS assume mais tarefas operacionais, enquanto a equipe configura domínios, validação, listeners e políticas.

A VPS oferece mais controle direto e costuma exigir mais manutenção manual. Os serviços gerenciados da AWS facilitam integração, renovação e escala, mas podem aumentar custo, complexidade de configuração e dependência da plataforma.

## Resumo

**TLS é o protocolo que protege dados em trânsito, ajuda a confirmar a identidade do servidor e detecta alterações na comunicação. HTTPS é HTTP usando TLS.**

Ele é essencial para uma aplicação segura, mas precisa ser combinado com autenticação, autorização, validação de entradas, proteção de segredos e boas práticas de [[Segurança]].
