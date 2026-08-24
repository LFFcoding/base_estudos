# O que é n8n?

**n8n** (pronunciado “en-eit-en”) é uma ferramenta de automação de workflows (*workflow automation*). Ela permite conectar aplicações, serviços e APIs, transformar dados e executar uma sequência de ações com pouco ou nenhum código.

O n8n é descrito pela documentação oficial como uma ferramenta de automação com licença *fair-code*. Ele pode ser usado na nuvem ou instalado pela própria equipe (*self-hosted*). Isso é diferente de dizer que ele é simplesmente uma [[Biblioteca]] ou um [[Framework]].

Uma analogia é uma linha de montagem: cada etapa recebe uma entrada, realiza uma tarefa e entrega o resultado à próxima etapa. No n8n, cada etapa costuma ser representada por um **node** (nó).

## O que é um workflow?

Um **workflow** é um fluxo de trabalho automatizado. Ele descreve o que inicia o processo, quais passos devem ser executados e o que acontece ao final.

Um exemplo:

```text
Webhook -> validar dados -> chamar API -> salvar resultado -> enviar notificação
```

Esse fluxo pode ser executado toda vez que um sistema enviar uma [[Requisição]] para o webhook.

## O que são nodes?

Os **nodes** são os blocos que formam um workflow. Cada node possui uma responsabilidade específica e normalmente recebe dados de uma etapa anterior.

Alguns tipos comuns são:

- **Trigger:** inicia o workflow;
- **HTTP Request:** chama uma API ou serviço externo;
- **Database:** consulta ou altera dados;
- **IF ou Switch:** escolhe um caminho conforme uma condição;
- **Transform:** altera, filtra ou organiza os dados;
- **Code:** executa uma transformação personalizada em JavaScript ou Python;
- **Merge:** combina dados de caminhos diferentes;
- **Respond to Webhook:** devolve uma resposta a quem iniciou o fluxo.

O n8n possui nodes para muitos serviços. Quando não existe um node específico, o node de HTTP Request pode chamar uma API diretamente. Uma requisição criada com [[cURL]] pode ajudar a entender quais método, URL, headers e corpo devem ser configurados.

## Como um workflow é iniciado?

Um workflow pode ser iniciado por diferentes tipos de gatilho (*triggers*):

- **Webhook:** outro sistema envia uma requisição para uma URL;
- **Schedule:** o n8n executa em horários ou intervalos definidos;
- **Evento de aplicação:** algo acontece em um serviço conectado;
- **Execução manual:** a pessoa executa o fluxo para testar;
- **Outro workflow:** um fluxo chama outro fluxo.

O gatilho deve ser escolhido de acordo com a origem do evento. Se um sistema já consegue avisar imediatamente por webhook, não é necessário consultar a mesma informação a cada minuto.

## Exemplo prático

Imagine uma loja que precisa avisar o setor de expedição quando um pedido for aprovado:

```text
1. Webhook recebe o pedido aprovado.
2. Node de validação confere os campos obrigatórios.
3. HTTP Request consulta detalhes no [[Backend]].
4. Node de decisão verifica se existe estoque.
5. Node de banco grava o registro no [[PostgreSQL]].
6. Node de mensagem envia um aviso para a equipe.
```

O n8n pode orquestrar esse processo, mas a regra principal de negócio deve continuar protegida no backend. Não é uma boa ideia permitir que um workflow contorne autorização, validação ou transações importantes da aplicação.

## n8n não é a mesma coisa que backend

Um [[Backend]] normalmente oferece regras de negócio, endpoints, autenticação e acesso controlado aos dados de um produto.

O n8n é mais voltado para integração e automação entre sistemas. Ele pode chamar o backend e reagir a eventos, mas não precisa substituir a aplicação principal.

Uma arquitetura possível é:

```text
Frontend -> [[Requisição]] -> Backend -> Banco de dados
                              ^
                              |
                         n8n automatiza integrações
```

Por exemplo, o backend pode registrar um pagamento e o n8n pode receber um evento para enviar um e-mail, atualizar um CRM ou gerar um documento.

## Credenciais

Para chamar uma API, o n8n precisa de uma credencial, como uma chave de API, usuário e senha, token, OAuth ou certificado.

As credenciais devem ser cadastradas no mecanismo próprio do n8n e referenciadas pelos nodes. Não coloque segredos diretamente em textos, expressões, nomes de workflows ou código exportado.

O acesso de cada credencial deve ser limitado ao que o workflow realmente precisa. Uma chave que apenas consulta pedidos não deveria poder apagar todos os registros.

## Dados dentro do workflow

Os dados passam de um node para o seguinte. Um node pode receber um objeto JSON como este:

```json
{
  "pedidoId": 42,
  "cliente": "Ana",
  "total": 159.90
}
```

O próximo node pode ler `pedidoId`, consultar o [[Backend]] e acrescentar o resultado ao mesmo fluxo.

Trate os dados com cuidado: valide o formato, limite o tamanho de entradas, remova informações desnecessárias e evite enviar dados pessoais para serviços que não precisam deles.

## n8n e APIs

O n8n é útil quando sistemas diferentes precisam conversar:

```text
Sistema A -> n8n -> API do Sistema B
```

O node de HTTP Request permite definir método, URL, headers, parâmetros, corpo e autenticação. Esses elementos fazem parte de uma [[Requisição]] e devem ser configurados de acordo com a documentação da API.

Se uma API usa [[OAuth 2.0]], [[OIDC]] ou outro mecanismo de autenticação, o workflow deve seguir o fluxo correto. Não transforme um token de acesso em texto fixo dentro de um node.

## n8n e mensageria

Um workflow também pode participar de uma arquitetura orientada a eventos. Ele pode consumir ou publicar mensagens em soluções como [[RabbitMQ]] ou trabalhar com [[Filas]].

Isso é diferente de apenas chamar uma API:

- uma chamada HTTP normalmente espera uma resposta imediata;
- uma fila permite que a mensagem seja processada depois;
- um broker pode distribuir mensagens para consumidores diferentes.

Ao processar mensagens, pense em duplicidade, tentativas, ordem, falhas e idempotência. A operação pode ser repetida, então use as orientações de [[Idempotência]] quando o processo alterar dados.

## n8n e banco de dados

O n8n pode consultar ou alterar um banco, mas conexões de produção precisam ser planejadas:

- use um usuário com permissões mínimas;
- não monte SQL concatenando dados recebidos de usuários;
- prefira parâmetros de consulta;
- limite o volume de registros retornados;
- trate transações e falhas de forma explícita;
- registre o que foi processado sem expor dados sensíveis.

Para grandes volumes, não use o n8n como substituto de um serviço especializado de processamento sem analisar filas, concorrência, tempo de execução e recuperação de falhas.

## Como executar o n8n?

As opções principais são:

### n8n Cloud

A equipe usa o serviço hospedado pelo fornecedor. Não precisa administrar o servidor, atualizações ou parte da infraestrutura, mas existe dependência do serviço e do plano contratado.

### Self-hosted

A equipe instala e administra o n8n. Ele pode ser executado com [[Docker]] e [[Docker Compose]], em uma máquina local, servidor ou [[VPS]].

Uma instalação própria exige cuidar de:

- armazenamento persistente;
- backups;
- atualizações;
- domínio e acesso externo;
- autenticação;
- [[TLS]];
- logs e monitoramento;
- capacidade e disponibilidade.

Um [[Caddy]] pode atuar como proxy reverso, receber HTTPS e encaminhar o tráfego para o container do n8n.

### Execução por npm

O n8n também pode ser instalado usando o ecossistema [[Node.js]] e npm. Essa opção pode ser útil para testes ou desenvolvimento, mas produção precisa de um plano para manter o processo ativo, reiniciá-lo e armazenar seus dados.

## VPS e AWS

Em uma [[VPS]], uma opção simples é executar n8n com Docker Compose, proteger o acesso com Caddy e fazer backup dos volumes e do banco utilizado.

Na AWS, a alternativa mais próxima é uma instância **EC2** executando Docker. A diferença é que a equipe continua administrando o sistema operacional, atualizações e disponibilidade, assim como em uma VPS.

Outra opção é executar o container em **ECS com Fargate**, reduzindo a administração do servidor. Isso não é uma correspondência exata: ainda é necessário configurar rede, armazenamento, banco, secrets, logs e escalabilidade. Serviços AWS como EventBridge, Step Functions e Lambda podem resolver partes da automação, mas têm modelos diferentes do editor visual e dos nodes do n8n.

## Segurança

O n8n pode acessar APIs, bancos, arquivos e serviços importantes. Por isso, uma instalação exposta na internet precisa ser tratada como um sistema de produção.

Boas práticas:

- use HTTPS com [[TLS]];
- proteja a interface administrativa com autenticação forte e, quando possível, SSO;
- não deixe webhooks sensíveis sem autenticação ou validação;
- use credenciais com privilégio mínimo;
- mantenha o n8n atualizado;
- não instale nodes de terceiros sem revisar a origem e as permissões;
- avalie cuidadosamente nodes que executam código ou acessam o sistema de arquivos;
- defina uma chave de criptografia e proteja as variáveis de ambiente;
- faça backup e teste a restauração;
- não registre tokens, senhas ou dados pessoais nos logs;
- separe ambientes de desenvolvimento, homologação e produção;
- use a auditoria do n8n para procurar credenciais sem uso, webhooks desprotegidos, nodes arriscados e configurações ausentes.

Compartilhar um workflow pode dar acesso às credenciais usadas por ele, de acordo com as permissões do ambiente. Compartilhe somente com pessoas e projetos que realmente precisam desse acesso.

## Confiabilidade e manutenção

Uma automação não deve apenas funcionar no caminho feliz. Planeje também:

- o que acontece quando a API está fora do ar;
- quantas tentativas serão feitas;
- como evitar duplicidade;
- como registrar o erro;
- quem será avisado;
- como retomar o processamento;
- quanto tempo uma execução pode consumir;
- como limitar chamadas para não ultrapassar o rate limit.

Workflows pequenos e com responsabilidades claras são mais fáceis de testar e manter. Quando um fluxo crescer demais, divida-o em sub-workflows ou mova regras importantes para um serviço de [[Backend]].

## Versionamento

Workflows devem ser tratados como código importante. Exporte ou versiona suas definições usando [[Git]] quando o processo da equipe permitir.

Nunca coloque credenciais reais no repositório. Também documente variáveis necessárias, dependências externas, formato de entrada, formato de saída e como reprocessar uma execução com falha.

O n8n possui recursos de controle de fonte e ambientes em determinados planos. Mesmo assim, é responsabilidade da equipe definir uma estratégia segura para desenvolvimento, aprovação e publicação.

## n8n e CI/CD

O n8n pode participar do [[CI-CD]], mas não é a mesma coisa que uma ferramenta de CI/CD.

- **n8n:** automatiza fluxos de negócio e integrações;
- **CI/CD:** valida, constrói e entrega software;
- **Git:** registra versões e alterações;
- **Backend:** implementa regras e serviços da aplicação.

Um exemplo é usar CI/CD para validar um workflow versionado e depois publicar uma versão em um ambiente n8n. A forma exata depende do plano e da estratégia de implantação.

## Quando usar n8n?

n8n pode ser uma boa escolha para:

- integrar aplicações que já possuem APIs;
- criar automações internas rapidamente;
- transformar e encaminhar dados;
- receber webhooks e disparar processos;
- criar protótipos de integrações;
- automatizar tarefas administrativas;
- conectar serviços sem escrever uma aplicação inteira para cada integração.

## Quando ter cuidado?

Avalie outra solução ou uma combinação com backend quando:

- a lógica de negócio for central e muito complexa;
- houver volume ou latência que exijam processamento especializado;
- o fluxo precisar de garantias transacionais fortes;
- a integração precisar de testes automatizados detalhados;
- a execução depender de código não confiável;
- uma indisponibilidade do n8n paralisar uma parte crítica do negócio;
- o workflow ficar tão grande que ninguém consegue revisá-lo.

O n8n pode fazer parte da solução, mas não elimina a necessidade de arquitetura, segurança, observabilidade e [[Testes]].

## Resumo

n8n é uma plataforma de automação visual que conecta aplicações e APIs por meio de workflows formados por nodes. Ele pode iniciar fluxos por webhook, agenda ou eventos, transformar dados e chamar serviços externos. É útil para integrações e automações, mas deve ser protegido, versionado e usado com cuidado quando envolver regras de negócio, dados sensíveis ou processos críticos.

### Veja também

- [[Backend]]
- [[Requisição]]
- [[cURL]]
- [[Docker]]
- [[Docker Compose]]
- [[Container]]
- [[VPS]]
- [[Caddy]]
- [[TLS]]
- [[PostgreSQL]]
- [[Redis]]
- [[RabbitMQ]]
- [[Filas]]
- [[Idempotência]]
- [[Git]]
- [[CI-CD]]
- [[Node.js]]
- [[JavaScript]]
- [[TypeScript]]
- [[Segurança]]
- [[Testes]]

### Referências oficiais

- [Documentação do n8n](https://docs.n8n.io/)
- [Página oficial do n8n](https://n8n.io/)
- [Recursos e funcionalidades](https://n8n.io/features/)
- [Auditoria de segurança](https://docs.n8n.io/hosting/securing/security-audit/)
- [Ambientes com controle de fonte](https://docs.n8n.io/source-control-environments/create-environments/)
