# cURL

**cURL** é uma ferramenta de linha de comando para transferir dados usando uma URL. O comando mais conhecido é `curl`, usado para fazer [[Requisição|requisições]] HTTP, consultar APIs, enviar dados, baixar arquivos e investigar conexões.

O nome vem de *Client URL*. Além do programa de [[Terminal]], existe a biblioteca **libcurl**, que permite incorporar essas transferências em outros programas.

## Uma analogia

Um navegador é como um atendente que prepara um pedido, envia para um endereço e apresenta o resultado de forma visual.

O cURL faz algo parecido, mas conversa diretamente pelo [[Terminal]] e mostra a resposta em texto. Isso é útil quando não há interface gráfica ou quando queremos controlar exatamente o método, os headers, o body e as opções da conexão.

## Verificar se está instalado

```bash
curl --version
```

O comando mostra a versão e os protocolos suportados. Em muitas distribuições Linux e sistemas macOS ele já está instalado. Em ambientes Windows, `curl.exe` pode ser usado para deixar claro que o programa cURL está sendo chamado, especialmente em versões antigas do [[PowerShell]].

## Fazer uma requisição simples

```bash
curl https://example.com
```

Esse comando faz uma requisição `GET` e imprime o corpo da resposta no terminal.

Para ver somente os headers:

```bash
curl --head https://example.com
```

Para acompanhar também os redirecionamentos:

```bash
curl --location https://example.com
```

- `--head` ou `-I` pede os headers;
- `--location` ou `-L` segue respostas de redirecionamento, como `301` e `302`.

## Consultar uma API

Uma API pode receber uma requisição com método, headers e body:

```bash
curl --request GET \
  --url https://api.exemplo.com/usuarios/42 \
  --header 'Accept: application/json'
```

O exemplo pede o usuário `42` e informa que o cliente aceita uma resposta em [[JSON]].

Para enviar JSON:

```bash
curl --request POST \
  --url https://api.exemplo.com/usuarios \
  --header 'Content-Type: application/json' \
  --header 'Accept: application/json' \
  --data '{"nome":"Ana","email":"ana@example.com"}'
```

- `--request POST` escolhe o método HTTP;
- `--url` informa o endereço;
- `--header` adiciona metadados à [[Requisição]];
- `--data` envia o body.

O [[Backend]] deve validar os dados recebidos e a autorização. O fato de uma requisição ter sido feita com cURL não torna os dados confiáveis.

## Enviar autenticação

Um access token pode ser enviado no header `Authorization`:

```bash
curl --request GET \
  --url https://api.exemplo.com/pedidos \
  --header 'Authorization: Bearer <access-token>'
```

`<access-token>` é apenas um placeholder. Nunca coloque um token real em documentação pública, histórico do terminal, scripts versionados ou logs.

Autenticação básica pode ser usada quando o serviço exigir:

```bash
curl --user '<usuario>:<senha>' https://api.exemplo.com/privado
```

Evite colocar credenciais reais diretamente no comando, porque elas podem aparecer no histórico ou na lista de processos. Prefira um gerenciador de segredos, variáveis protegidas ou outra forma segura de fornecimento.

## Ver o status e o tempo da resposta

Para mostrar o código de status ao final:

```bash
curl --silent --show-error \
  --output /dev/null \
  --write-out 'status=%{http_code} tempo=%{time_total}s\n' \
  https://example.com
```

- `--silent` reduz mensagens de progresso;
- `--show-error` mantém erros importantes visíveis;
- `--output /dev/null` descarta o corpo;
- `--write-out` imprime informações escolhidas da resposta.

Para fazer o comando falhar quando o servidor responder com erro HTTP:

```bash
curl --fail-with-body --silent --show-error https://api.exemplo.com/saude
```

Isso é útil em scripts e verificações de saúde no [[CI-CD]]. Ainda assim, um status HTTP bem-sucedido não garante que o conteúdo esteja correto.

## Baixar arquivos

Para salvar a resposta em um arquivo escolhido:

```bash
curl --output relatorio.pdf https://example.com/relatorio.pdf
```

Para usar o nome sugerido pela URL:

```bash
curl --remote-name https://example.com/relatorio.pdf
```

- `--output` ou `-o` escolhe o nome do arquivo;
- `--remote-name` ou `-O` usa o nome da URL.

Confirme o domínio e o tipo de arquivo antes de executar downloads em scripts. Um arquivo baixado não deve ser executado automaticamente sem validação.

## Debug e TLS

Para investigar uma conexão, o modo detalhado mostra etapas da comunicação:

```bash
curl --verbose https://example.com
```

Use `--verbose` com cuidado: headers podem conter cookies, tokens e outros dados sensíveis. Não compartilhe a saída sem revisar.

O cURL valida certificados TLS por padrão. Para indicar uma CA específica:

```bash
curl --cacert autoridade.pem https://api.exemplo.com
```

Não use `--insecure` ou `-k` para ignorar a validação TLS em produção:

```bash
# Evite em produção: desativa a validação do certificado.
curl --insecure https://api.exemplo.com
```

Isso pode esconder um certificado inválido e permitir um ataque de servidor intermediário. Veja mais em [[TLS]].

## Limites e timeouts

Um script não deve esperar indefinidamente por uma resposta:

```bash
curl --connect-timeout 5 \
  --max-time 20 \
  --fail-with-body \
  https://api.exemplo.com/saude
```

- `--connect-timeout` limita o tempo para conectar;
- `--max-time` limita o tempo total;
- `--fail-with-body` retorna erro para status HTTP de falha sem esconder o corpo da resposta.

Em chamadas que podem ser repetidas, configure tentativas com cuidado. Repetir automaticamente um `POST` pode criar dados duplicados se a operação não for [[Idempotência|idempotente]].

## cURL em scripts e no CI/CD

cURL pode verificar se um serviço está disponível:

```bash
curl --fail-with-body --silent --show-error \
  --retry 3 \
  --retry-delay 2 \
  https://api.exemplo.com/health
```

Esse tipo de comando pode ser usado no [[CI-CD]], em testes de fumaça e em diagnósticos de um serviço executado em [[Docker]] ou em uma [[VPS]]. O endpoint de saúde deve revelar somente o necessário e não deve expor senhas, tokens ou detalhes internos.

## cURL, navegador e ferramentas gráficas

- o navegador facilita visualizar páginas e interagir com telas;
- o cURL permite repetir requisições de forma rápida e automatizada;
- ferramentas gráficas ajudam a organizar coleções, variáveis e respostas;
- bibliotecas da aplicação permitem fazer requisições dentro do código.

O cURL não substitui os [[Testes]] da aplicação. Ele ajuda a verificar o comportamento observado pela rede e é especialmente útil para investigar uma [[Requisição]] isolada.

## Boas práticas

- use `--silent --show-error` em scripts para controlar a saída;
- defina timeout de conexão e tempo total;
- verifique o status HTTP e, quando necessário, valide o conteúdo da resposta;
- use HTTPS e mantenha a validação TLS ativada;
- não coloque segredos em comandos, arquivos versionados ou logs;
- use placeholders em exemplos públicos;
- codifique corretamente URLs, query parameters e bodies;
- escape dados quando gerar comandos automaticamente;
- evite repetir métodos que alteram dados sem garantir [[Idempotência]];
- revise comandos copiados da internet antes de executá-los;
- combine cURL com as práticas de [[Segurança]] e com o contrato da API.

**cURL é uma ferramenta de terminal para fazer transferências e requisições usando URLs, oferecendo controle detalhado sobre a comunicação e sendo muito útil para testes, automação e diagnóstico.**
