# O que é Agentic Systems Engineering?

**Agentic Systems Engineering** (*engenharia de sistemas agênticos*) é a prática de projetar, construir, testar, proteger e operar sistemas de software que usam agentes de inteligência artificial para realizar tarefas em várias etapas.

Um agente pode interpretar um objetivo, decidir qual ação tomar, usar ferramentas — como APIs, pesquisas ou funções da aplicação — e ajustar o próximo passo com base no resultado. O sistema completo inclui mais do que o modelo de IA: também inclui o software que controla as ações, integrações, dados, permissões, verificações, limites e mecanismos de supervisão.

A expressão ainda é usada de maneiras diferentes. Nesta nota, ela significa **engenharia de sistemas que contêm agentes**. Às vezes, a expressão também é usada para falar de **agentes que ajudam a executar atividades de engenharia de sistemas**, como analisar requisitos e rastreabilidade. São assuntos relacionados, mas não idênticos.

## Um agente não é apenas um modelo

Um modelo de linguagem pode produzir uma resposta a partir de uma entrada. Um agente é parte de um sistema que pode usar esse modelo para escolher ações e interagir com um ambiente. Uma aplicação agêntica normalmente combina:

- **Modelo e instruções:** interpretam o objetivo e ajudam a selecionar o próximo passo;
- **Orquestração:** controla o ciclo de execução, as etapas, os limites e quando encerrar;
- **Ferramentas:** funções controladas para consultar ou alterar dados e chamar outros serviços;
- **Contexto e estado:** informações necessárias para dar continuidade à tarefa; podem incluir histórico ou recuperação de conhecimento, como em [[RAG]];
- **Políticas e controles:** determinam quais ações são permitidas e quando uma pessoa precisa aprová-las;
- **Avaliação e observabilidade:** medem os resultados e ajudam a investigar o que ocorreu em cada execução.

O modelo pode sugerir uma ação, mas é a aplicação que deve validar e executar a chamada à ferramenta conforme as permissões definidas. Essa separação permite aplicar regras convencionais de software ao redor de um componente probabilístico.

## Workflow ou agente?

Um **workflow** usa etapas definidas previamente pelo código. Por exemplo: classificar uma mensagem, buscar dados e montar uma resposta, sempre nessa ordem.

Um **agente** pode escolher dinamicamente quais etapas e ferramentas usar para atingir o objetivo. Ele pode repetir uma ação, mudar o plano diante de um resultado inesperado ou pedir ajuda a uma pessoa.

Um workflow costuma ser mais previsível para processos conhecidos e repetitivos. Um agente pode ser útil quando os passos necessários variam de acordo com a situação. Nem todo sistema com IA precisa ser agêntico; a autonomia adicional também pode aumentar custo, latência e risco de erros acumulados.

## O que precisa ser projetado?

### Objetivo e limites

Defina qual tarefa o sistema deve concluir, quais resultados contam como sucesso e quais situações devem interromper a execução. Uma meta vaga, como “resolva o problema do cliente”, é difícil de avaliar e pode dar margem a ações inesperadas. É melhor declarar o escopo, os dados que podem ser usados e o que o agente não tem permissão para fazer.

### Ferramentas e integrações

Cada ferramenta é uma capacidade concedida ao agente. Uma ferramenta bem projetada tem propósito claro, parâmetros validados, resposta estruturada e permissões limitadas. Prefira ferramentas específicas, como `consultar_pedido`, a uma função genérica que permita executar qualquer comando ou consulta.

As ferramentas devem validar a entrada e a autorização no código da aplicação, sem depender apenas de instruções em linguagem natural. Para operações que possam ser repetidas após uma falha ou timeout, considere [[Idempotência]] para evitar efeitos duplicados.

### Autonomia e aprovação humana

Decida o que o sistema pode fazer sozinho. Uma progressão comum é:

1. consultar informações;
2. preparar uma sugestão ou rascunho;
3. executar uma ação reversível e de baixo impacto;
4. solicitar aprovação antes de ações sensíveis, irreversíveis ou com impacto financeiro.

A aprovação deve mostrar à pessoa o que será feito e sobre quais dados. Também é importante permitir interrupção, definir limites de tentativas e ter uma forma de recuperar-se de ações incorretas.

### Contexto e memória

O sistema precisa decidir quais informações entram no contexto do agente, por quanto tempo permanecem disponíveis e quem pode acessá-las. Histórico de conversa não deve ser tratado automaticamente como fonte confiável ou memória permanente. Dados recuperados de documentos e ferramentas também precisam ser delimitados e tratados de acordo com sua origem e sensibilidade.

### Avaliação e observabilidade

Avalie o comportamento de ponta a ponta, não apenas se uma resposta parece bem escrita. Use cenários representativos para verificar se o agente escolhe a ferramenta adequada, respeita limites, reconhece quando não sabe e termina a tarefa corretamente. Inclua casos de erro, instruções conflitantes e tentativas de provocar ações indevidas.

Em produção, registre informações suficientes para depurar e auditar a execução — por exemplo, ferramentas chamadas, resultados, aprovações e motivo de encerramento. Proteja esses registros: remova dados pessoais ou segredos desnecessários e defina acesso e retenção. Acompanhe também taxa de sucesso, falhas, custo, latência e escalonamentos para pessoas.

## Ciclo de engenharia

Um processo prático pode seguir estas etapas:

1. **Identificar o problema:** confirme que a tarefa realmente exige flexibilidade; compare com uma solução comum de software ou um workflow fixo.
2. **Definir requisitos e riscos:** determine usuários, dados, ações possíveis, critérios de sucesso e danos que precisam ser evitados.
3. **Escolher o nível de autonomia:** decida o que pode ser lido, sugerido ou executado e quais ações dependem de aprovação.
4. **Projetar as ferramentas e o fluxo:** aplique validação, permissões mínimas, limites de tempo e de tentativas e condições explícitas de parada.
5. **Testar e avaliar:** use exemplos reais anonimizados ou cenários controlados, faça testes de segurança e meça regressões antes de liberar mudanças.
6. **Implantar gradualmente:** comece com acesso restrito ou modo de sugestão, observe os resultados e amplie as permissões somente com evidência.
7. **Monitorar e revisar:** acompanhe métricas, incidentes e mudanças no uso; atualize testes, ferramentas e regras quando o contexto mudar.

Esse ciclo combina práticas de engenharia de software, [[Testes]], [[Segurança]] e [[DevOps]] com avaliações específicas do comportamento do agente.

## Exemplo: agente de atendimento

Imagine um agente que ajuda a responder dúvidas sobre pedidos. Ele pode consultar o pedido, buscar a política de entrega e preparar uma resposta. Se a solução envolver um reembolso, o sistema pode preparar a solicitação, mas exigir confirmação de uma pessoa antes de executar a operação.

Nesse exemplo, a qualidade não depende só do texto final. Também importa se o agente consultou o pedido correto, usou uma política atual, não revelou dados de outra pessoa e respeitou a aprovação antes de alterar o sistema.

## Segurança e responsabilidade

Agentes podem ter acesso a dados e ferramentas capazes de alterar sistemas externos. Aumentar a autonomia sem limitar os poderes aumenta o impacto potencial de erros ou instruções maliciosas. Boas práticas incluem:

- conceder apenas o acesso necessário para a tarefa;
- restringir ferramentas e destinos a uma lista autorizada;
- validar autorização e parâmetros no backend antes de executar uma ação;
- pedir aprovação humana para operações de alto impacto;
- impor limites de tempo, custo, chamadas e tentativas;
- testar em ambiente isolado antes de dar acesso a sistemas reais;
- registrar decisões e ações sem armazenar segredos ou dados sensíveis desnecessários;
- oferecer meios de interromper o agente e tratar falhas.

Instruções do modelo ajudam a orientar o comportamento, mas não substituem controles técnicos. Regras essenciais — como acesso a dados, limites financeiros e aprovação — devem ser aplicadas pela aplicação e pela infraestrutura.

## Quando não usar agentes

Se o processo tem passos fixos, entradas conhecidas e regras determinísticas, uma função convencional ou workflow pode ser mais simples, barato e fácil de testar. Agentes são mais justificáveis quando há variação real de caminhos e quando a capacidade de interpretar contexto e escolher passos traz benefício mensurável.

Antes de adicionar autonomia, compare o ganho esperado com a complexidade de testar, proteger, monitorar e dar suporte ao sistema. Comece pela solução mais simples que atende aos requisitos.

## Relação com outros assuntos

- [[LangChain4j]] é um framework Java usado para integrar aplicações com modelos de linguagem e recursos relacionados;
- [[RAG]] descreve uma técnica que pode fornecer contexto recuperado ao sistema;
- [[Testes]] ajudam a validar componentes, integrações e cenários completos do agente;
- [[Segurança]] cobre controles de acesso, proteção de dados e redução de riscos;
- [[DevOps]] aborda implantação, operação e monitoramento contínuos;
- [[CI-CD]] pode automatizar verificações e avaliações antes de publicar mudanças.

## Referências

- [OpenAI: Building agents](https://developers.openai.com/tracks/building-agents) — ferramentas, orquestração, guardrails e tracing;
- [Anthropic: Building effective agents](https://www.anthropic.com/research/building-effective-agents) — diferença entre workflows e agentes, padrões de composição e uso criterioso de autonomia;
- [IMDA: Model AI Governance Framework for Agentic AI](https://www.imda.gov.sg/resources/press-releases-factsheets-and-speeches/press-releases/2026/new-model-ai-governance-framework-for-agentic-ai) — limites de poderes, supervisão humana e controles ao longo do ciclo de vida;
- [NIST: AI Risk Management Framework](https://airc.nist.gov/airmf-resources/airmf/) — gestão de riscos e confiança durante o projeto, desenvolvimento, uso e avaliação de sistemas de IA;
- [What Is Agentic Systems Engineering?](https://www.trace.space/blog/what-is-agentic-systems-engineering) — exemplo do uso da expressão para agentes que executam atividades de engenharia de sistemas.
