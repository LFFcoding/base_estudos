# O que é Agtech?

**Agtech** é a abreviação de **agricultural technology** (*tecnologia agrícola*). Também é comum encontrar o termo **agritech**. Ele representa o uso de tecnologia para resolver problemas da agricultura e da pecuária, melhorar decisões e tornar a produção mais eficiente, sustentável e rastreável.

Agtech não é um programa ou uma ferramenta específica. É um setor que combina conhecimento do campo com áreas como software, sensores, dados, automação, imagens, máquinas e conectividade.

Uma analogia: uma fazenda é como uma grande empresa a céu aberto. O clima, o solo, os animais, as máquinas e as pessoas produzem sinais o tempo todo. Uma solução Agtech funciona como um sistema nervoso: coleta esses sinais, organiza as informações e ajuda alguém a decidir qual ação tomar.

## Onde a Agtech pode ser usada?

As soluções podem acompanhar várias etapas da cadeia agrícola:

- **planejamento:** escolha de culturas, áreas e épocas de plantio;
- **produção:** acompanhamento de umidade, nutrientes, pragas, saúde animal e condições do campo;
- **operação:** controle de máquinas, insumos, irrigação, alimentação e tarefas;
- **gestão:** custos, produtividade, estoque, equipes e talhões;
- **pós-colheita:** armazenamento, qualidade, perdas e validade;
- **logística:** transporte, temperatura, localização e rastreabilidade;
- **comercialização:** origem do produto, contratos, preços e relacionamento com clientes.

O objetivo não é usar tecnologia por moda. Uma boa solução deve responder a uma necessidade real do produtor, da indústria, da cooperativa ou de outra pessoa da cadeia.

## Exemplos de soluções Agtech

### Agricultura de precisão

A agricultura de precisão usa dados para tratar partes diferentes da propriedade de maneira adequada. Em vez de aplicar a mesma quantidade de água ou fertilizante em toda a área, o sistema pode recomendar quantidades conforme o solo, o relevo e o histórico de cada região.

Isso pode reduzir desperdícios e melhorar o uso de recursos. A recomendação ainda precisa considerar o conhecimento agronômico e ser validada no campo.

### Monitoramento por sensores

Sensores podem medir umidade do solo, temperatura, chuva, nível de água, localização ou condições de um equipamento. Leituras frequentes ajudam a perceber mudanças que seriam difíceis de acompanhar manualmente.

Um sensor não transforma automaticamente uma medição em uma boa decisão. É preciso verificar sua calibração, sua posição, a unidade usada e a qualidade da comunicação.

### Gestão de rebanhos

Uma solução para pecuária pode registrar animais, movimentações, vacinas, pesagens, alimentação, reprodução e ocorrências de saúde. Esses dados apoiam o manejo e a rastreabilidade.

O sistema deve controlar quem pode acessar informações sensíveis e manter o histórico das alterações. Um dado corrigido sem registro pode prejudicar decisões e auditorias.

### Imagens e mapas

Drones, satélites e câmeras podem produzir imagens para identificar falhas no plantio, estresse hídrico, áreas com pragas ou mudanças no terreno. As imagens precisam ser interpretadas com cuidado, pois sombra, nuvem, iluminação e qualidade do equipamento podem alterar o resultado.

### Rastreabilidade

Rastreabilidade (*traceability*) é a capacidade de acompanhar a origem, as transformações e o caminho de um produto. Um registro pode relacionar lote, talhão, animal, insumo, data, operação e destino.

Além de ajudar na qualidade, a rastreabilidade pode apoiar auditorias, certificações e respostas rápidas quando surge um problema.

## Como uma solução Agtech funciona?

Um fluxo simplificado pode ser representado assim:

```text
sensor ou aplicativo
        ↓
conectividade e envio dos dados
        ↓
backend: validação e regras de negócio
        ↓
banco de dados e armazenamento de históricos
        ↓
análise, alertas e relatórios
        ↓
frontend usado no celular ou computador
        ↓
decisão e ação no campo
```

Na prática:

1. um sensor mede a umidade do solo;
2. o dispositivo envia uma leitura;
3. o [[Backend]] valida o identificador, a unidade e o horário;
4. a leitura é salva no [[PostgreSQL]] ou em outro armazenamento adequado;
5. uma regra verifica se a umidade está abaixo do limite;
6. o sistema registra um alerta ou envia uma notificação;
7. a pessoa acompanha o resultado pelo [[Frontend]].

O envio pode usar uma [[Requisição]] HTTP ou [[Protocolos de mensageria]]. Quando a conexão estiver instável, uma [[Filas|fila]] pode guardar os dados até que o processamento seja possível.

## Exemplo de leitura de sensor

Um dispositivo poderia enviar um payload em [[JSON]] como este:

```json
{
  "sensorId": "sensor-023",
  "talhaoId": "talhao-12",
  "umidadeSolo": 31.4,
  "unidade": "%",
  "medidoEm": "2026-10-13T09:30:00Z"
}
```

O backend não deve confiar cegamente no valor recebido. Ele pode verificar se:

- o sensor existe e pertence à propriedade correta;
- a pessoa ou o dispositivo tem autorização para enviar dados;
- a umidade está dentro de um intervalo possível;
- a unidade esperada é realmente `%`;
- o horário é válido e não está muito distante do horário atual;
- aquela mesma leitura não foi processada duas vezes.

Uma chamada simples poderia ser feita com [[cURL]]:

```bash
curl -X POST https://agtech.exemplo.com/api/leituras \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "sensorId": "sensor-023",
    "talhaoId": "talhao-12",
    "umidadeSolo": 31.4,
    "unidade": "%",
    "medidoEm": "2026-10-13T09:30:00Z"
  }'
```

O comando envia uma requisição HTTP para o endpoint. Em produção, o token não deve ser escrito diretamente em scripts versionados, terminais compartilhados ou arquivos públicos.

## Dados de uma Agtech

Uma aplicação agrícola pode lidar com diferentes tipos de dados:

- **dados cadastrais:** propriedades, talhões, animais, equipamentos e pessoas;
- **dados de eventos:** plantio, aplicação, vacinação, colheita e manutenção;
- **séries temporais:** medições repetidas ao longo do tempo;
- **dados espaciais:** pontos, limites, rotas e áreas no mapa;
- **imagens e arquivos:** fotos, laudos, notas e documentos;
- **dados derivados:** médias, alertas, previsões e indicadores.

O tipo de dado influencia a forma de armazenar e consultar as informações. O [[PostgreSQL]] pode atender bem a dados relacionais e, com extensões apropriadas, a dados espaciais. Um banco [[NoSQL]] ou um armazenamento de objetos pode ser útil em alguns cenários, mas a escolha deve partir dos requisitos.

Não é necessário guardar toda medição para sempre. Defina políticas de retenção, agregação e arquivamento conforme o valor do histórico, o custo e as exigências do negócio.

## Stack de software para uma Agtech

Uma aplicação da stack deste estudo poderia ser organizada assim:

- [[Java]] 17, [[Quarkus]] e [[Maven]] no backend;
- [[PostgreSQL]] para cadastros, operações e histórico relacional;
- [[Redis]] para dados temporários, sessões, limites ou [[Cache]] quando fizer sentido;
- [[Filas]] e [[RabbitMQ]] para desacoplar ingestão, processamento e notificações;
- [[Next.js]] e [[React]] para dashboards e telas de operação;
- [[Docker Compose]] para levantar a aplicação, o banco e os serviços auxiliares durante o desenvolvimento;
- [[Caddy]] como proxy reverso e terminador TLS;
- [[VPS]] para um ambiente simples de hospedagem.

Essa composição é apenas uma possibilidade. Se a quantidade de sensores, usuários ou dados aumentar muito, talvez seja necessário separar serviços, usar processamento em lote, adotar armazenamento especializado ou contratar serviços gerenciados.

## Conectividade no campo

Áreas rurais podem ter internet lenta, instável ou indisponível. Uma aplicação Agtech precisa considerar que o dispositivo ou o aplicativo ficará sem conexão em algum momento.

Estratégias possíveis:

- armazenar leituras localmente no dispositivo;
- enviar os dados novamente quando a conexão voltar;
- mostrar claramente quando uma informação ainda não foi sincronizada;
- registrar o horário da medição separado do horário do envio;
- usar mensagens com identificadores únicos;
- manter a operação essencial disponível offline;
- evitar enviar imagens grandes quando uma informação menor for suficiente.

O backend precisa aceitar que as mensagens podem chegar atrasadas ou fora de ordem. Uma leitura feita às 9h pode chegar depois de outra feita às 10h, e o sistema deve usar o horário da medição para interpretar a sequência.

## Idempotência e duplicidade

Sensores e aplicativos podem repetir o envio quando não sabem se a primeira tentativa chegou ao servidor. Se o backend criar dois registros para a mesma medição, os indicadores ficarão errados.

Uma solução é usar uma [[Chave de idempotência]] ou um identificador único da leitura. O processamento deve ser uma [[Idempotência|operação idempotente]]: repetir a mesma mensagem não pode produzir efeitos duplicados.

Exemplo de identificador:

```json
{
  "leituraId": "sensor-023-20261013T093000Z",
  "sensorId": "sensor-023",
  "umidadeSolo": 31.4
}
```

O banco pode criar uma restrição de unicidade para `leituraId`. Mesmo que a mensagem seja recebida duas vezes, somente uma gravação será aceita.

## Alertas e processamento assíncrono

Nem todo processamento precisa acontecer durante a requisição. O backend pode salvar a leitura rapidamente e enviar uma mensagem para um worker analisar limites, gerar indicadores ou notificar alguém.

Um fluxo possível é:

```text
API recebe leitura
        ↓
salva leitura e evento pendente
        ↓
broker de fila
        ↓
worker analisa a regra
        ↓
cria alerta e atualiza dashboard
```

Um [[Broker de fila]] como o [[RabbitMQ]] ajuda a distribuir mensagens entre produtores e consumidores. É importante definir tentativas, tempo de espera, mensagens que não puderam ser processadas e uma forma de observar falhas.

O processamento assíncrono melhora a resiliência, mas torna o sistema eventualmente consistente: o dashboard pode demorar alguns segundos para mostrar um alerta. Essa característica deve ser conhecida por quem usa o sistema.

## Segurança

Uma Agtech pode guardar informações sobre propriedades, produção, localização, animais, pessoas e operações. Esses dados merecem proteção.

Boas práticas:

- autenticar dispositivos, pessoas e integrações;
- autorizar o acesso por propriedade, organização e função;
- usar [[TLS]] na comunicação;
- armazenar senhas e tokens de forma segura;
- não colocar segredos no código ou nos logs;
- validar dados recebidos antes de processá-los;
- registrar quem alterou informações importantes;
- limitar a quantidade de requisições para evitar abuso;
- revogar dispositivos perdidos ou comprometidos;
- criar backups protegidos e testar a restauração.

O uso de [[JWT]] pode fazer parte da autenticação de uma API, mas o token sozinho não decide se a pessoa pode acessar uma propriedade específica. A autorização precisa verificar o contexto do recurso.

Também é importante pensar na segurança física: um dispositivo instalado no campo pode ser roubado, adulterado ou ficar exposto ao clima. O sistema deve tratar as leituras como dados potencialmente não confiáveis.

## Qualidade dos dados

Uma tela bonita não corrige dados ruins. Antes de criar gráficos, verifique:

- calibração e manutenção dos sensores;
- unidade e precisão das medições;
- horário e fuso usados;
- leituras ausentes ou repetidas;
- valores fora dos limites possíveis;
- alterações manuais e sua justificativa;
- origem e confiabilidade de cada dado.

Um alerta também deve explicar por que foi criado. “Umidade baixa no talhão 12 às 9h” é mais útil do que apenas “há um problema”. Quando possível, mostre o valor medido, o limite configurado e o horário da leitura.

## Testes e operação

Uma solução Agtech precisa funcionar em situações variadas. Os [[Testes]] podem verificar:

- validação de payloads e unidades;
- duplicidade de leituras;
- mensagens fora de ordem;
- perda temporária de conexão;
- reprocessamento após uma falha;
- permissões entre propriedades diferentes;
- cálculos de médias, limites e indicadores;
- comportamento quando o sensor fica sem enviar dados.

Testes de carga podem simular muitos sensores enviando leituras ao mesmo tempo. Testes de integração devem verificar o banco, o broker e os serviços reais ou equivalentes. O [[DevOps]] ajuda a automatizar essas verificações e acompanhar o sistema depois da publicação.

## Agtech na VPS e na AWS

Em uma [[VPS]], a equipe pode executar o backend, o frontend, o banco e os workers com [[Docker Compose]]. Isso oferece controle e costuma ser simples para um ambiente pequeno, mas exige administrar atualizações, backups, firewall, monitoramento, armazenamento e capacidade do servidor.

A AWS não possui um único serviço chamado “Agtech”. Ela oferece peças gerenciadas que podem formar uma arquitetura agrícola:

- **AWS IoT Core:** conecta dispositivos e recebe mensagens;
- **API Gateway:** expõe APIs HTTP;
- **Amazon RDS for PostgreSQL:** administra um banco PostgreSQL gerenciado;
- **Amazon S3:** armazena imagens, documentos e arquivos;
- **Amazon ECS ou Fargate:** executa containers sem administrar diretamente todos os servidores;
- **Amazon Timestream:** pode armazenar algumas séries temporais;
- **Amazon CloudWatch:** reúne logs, métricas e alertas;
- **IAM:** controla permissões entre pessoas e serviços.

As principais diferenças são:

- na VPS, a equipe administra mais componentes diretamente e paga por um servidor definido;
- na AWS, os serviços gerenciados reduzem parte da operação, mas aumentam a quantidade de configurações e podem elevar o custo;
- a AWS oferece mais opções para escalar e conectar dispositivos, enquanto a VPS pode ser suficiente para poucos usuários e sensores;
- em ambos os casos, é necessário proteger credenciais, testar backups, monitorar falhas e controlar custos.

Uma decisão desse tipo pode ser registrada em um [[ADR]] para explicar volume esperado, orçamento, riscos e motivos da escolha.

## Boas práticas

- começar com um problema mensurável, como reduzir desperdício ou melhorar rastreabilidade;
- conversar com as pessoas que trabalham no campo antes de definir telas e alertas;
- projetar funcionamento offline e sincronização posterior;
- registrar unidade, origem, horário e qualidade das medições;
- separar horário da medição do horário de recebimento;
- usar identificadores únicos e processamento idempotente;
- validar e monitorar dados antes de gerar recomendações;
- permitir correções manuais com histórico e justificativa;
- proteger dados, dispositivos, APIs e credenciais;
- testar falhas de rede, duplicidade, atraso e indisponibilidade;
- evitar dashboards cheios de gráficos que não ajudam em nenhuma decisão;
- medir se a tecnologia realmente melhorou o resultado esperado;
- documentar decisões importantes com [[ADR]] e manter o sistema versionado no [[Git]].

## Resumo

> **Agtech é o uso de tecnologia para melhorar decisões, operações e resultados na agricultura e na pecuária.**

Uma boa solução conecta dados do campo a ações úteis. Ela combina conhecimento agronômico, software, conectividade, segurança e operação confiável, sempre lembrando que a tecnologia deve apoiar as pessoas e não substituir automaticamente o conhecimento de quem trabalha no campo.
