# O que é LangChain4j?

**LangChain4j** é uma biblioteca Java para criar aplicações que usam modelos de linguagem (*Large Language Models*, ou LLMs). Ela oferece APIs para conversar com modelos, manter memória de conversas, estruturar respostas, consultar documentos, chamar funções do código Java e construir aplicações com IA.

Uma analogia: o modelo de linguagem é o motor de uma máquina, enquanto o LangChain4j é o conjunto de adaptadores, controles e ferramentas que permite colocar esse motor dentro de uma aplicação Java de forma organizada.

O LangChain4j não é um modelo de IA. Ele também não hospeda automaticamente o modelo. Ele ajuda o código Java a conversar com provedores de modelos e a organizar o fluxo ao redor deles.

## Para que ele serve?

Com LangChain4j, uma aplicação pode:

- enviar prompts para modelos de chat;
- receber respostas de texto ou estruturas definidas;
- manter parte do histórico de uma conversa;
- transformar documentos em embeddings;
- buscar informações relevantes antes de responder;
- permitir que o modelo solicite uma função Java;
- integrar modelos e bancos de dados vetoriais;
- criar assistentes, chatbots, busca inteligente e fluxos com agentes.

O projeto possui integrações com diversos modelos e lojas de vetores por meio de APIs Java unificadas. A aplicação continua responsável por escolher o provedor, configurar limites, tratar falhas e proteger os dados.

## Relação com Java, Maven e Quarkus

LangChain4j foi feito para o ecossistema [[Java]] e pode ser incluído como dependência de um projeto [[Maven]]. A documentação oficial indica JDK 17 como versão mínima suportada atualmente, o que combina com a stack principal deste repositório.

Ele pode ser usado diretamente em uma aplicação Java ou integrado a frameworks como [[Quarkus]]. A integração com Quarkus pode fornecer serviços de IA injetáveis, configuração do framework, observabilidade e recursos de desenvolvimento.

Uma aplicação [[Backend]] Java pode usar LangChain4j dentro de um endpoint:

```text
Requisição do usuário
        ↓
endpoint do Backend
        ↓
serviço Java com LangChain4j
        ↓
modelo de linguagem
        ↓
resposta validada para o usuário
```

## Conceitos principais

### Chat model

O **chat model** é o componente que envia mensagens para um modelo de linguagem e recebe uma resposta.

Exemplo conceitual:

```java
ChatModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .build();

String resposta = model.chat("Explique o que é uma API.");
```

O exemplo usa uma integração de provedor. O nome do modelo, a dependência e as configurações devem ser escolhidos conforme o provedor usado no projeto.

A chave de API não deve ser colocada diretamente no código. Use variáveis de ambiente, secret managers ou a configuração segura do ambiente de execução.

### Prompt

Um **prompt** é a entrada enviada ao modelo. Ele pode conter instruções, contexto, dados do usuário e o formato esperado para a resposta.

Um prompt melhor costuma deixar claro:

- qual é o papel do assistente;
- qual tarefa deve ser feita;
- quais informações podem ser usadas;
- o que fazer quando não houver informação suficiente;
- qual formato deve ser devolvido;
- quais ações são proibidas.

Prompt não é uma barreira de segurança. Uma pessoa pode tentar enviar instruções que contradizem o objetivo da aplicação. Por isso, autorização, validação e regras importantes devem ficar no código.

### AI Services

**AI Services** permitem declarar uma interface Java que representa o serviço de IA. O LangChain4j cria a implementação e conecta a interface ao modelo configurado.

```java
interface Assistente {
    String responder(String pergunta);
}

Assistente assistente = AiServices.builder(Assistente.class)
    .chatModel(model)
    .build();

String resposta = assistente.responder("O que é PostgreSQL?");
```

Essa abordagem pode deixar o código mais legível, mas não elimina a necessidade de tratar erros, limites, validação e segurança.

### Memória de conversa

O modelo normalmente não lembra conversas anteriores sozinho. A aplicação precisa enviar mensagens anteriores ou usar um componente de **chat memory**.

Uma memória pode guardar as últimas mensagens, resumir o histórico ou buscar informações persistidas. Guardar tudo indefinidamente aumenta custo, latência e risco de expor dados.

Uma aplicação pode usar [[Redis]] ou outro armazenamento para conservar uma memória, mas deve definir expiração, privacidade e o identificador correto da conversa.

### Embeddings

Um **embedding** representa o significado aproximado de um texto como um vetor de números. Textos semanticamente parecidos tendem a ficar próximos em um espaço vetorial.

Isso permite buscar documentos por significado, e não somente por palavras iguais.

```text
"Como redefinir minha senha?"
                 ↓
       vetor numérico do texto
                 ↓
buscar trechos semanticamente parecidos
```

O modelo usado para criar os embeddings deve ser compatível com o modelo usado nas consultas. Também é importante escolher uma dimensão, uma estratégia de divisão dos documentos e uma forma de armazenar os vetores.

## RAG

**RAG** (*Retrieval-Augmented Generation*) significa geração aumentada por recuperação. Antes de pedir uma resposta ao modelo, a aplicação busca trechos relevantes em seus próprios dados e coloca esses trechos no prompt.

O processo normalmente possui duas partes:

### Indexação

1. carregar os documentos;
2. limpar e dividir o conteúdo em partes menores;
3. gerar embeddings para cada parte;
4. armazenar textos, metadados e vetores.

### Recuperação

1. receber a pergunta;
2. gerar o embedding da pergunta;
3. buscar trechos semelhantes;
4. enviar a pergunta com o contexto ao modelo;
5. devolver a resposta e, quando possível, as fontes usadas.

```text
documentos -> dividir -> embeddings -> armazenamento vetorial

pergunta -> embedding -> busca de trechos -> prompt -> modelo -> resposta
```

O RAG pode reduzir respostas inventadas quando os documentos recuperados são bons, mas não garante verdade. A aplicação deve lidar com documentos desatualizados, permissões, trechos irrelevantes e ausência de resultados.

O LangChain4j oferece opções de RAG simples, básicas e avançadas. Começar com uma versão simples pode ajudar em um protótipo; aplicações importantes precisam medir a qualidade e ajustar o processo.

## Tools e function calling

Um modelo pode solicitar que a aplicação execute uma função conhecida como **tool** ou **function calling**.

Por exemplo, a aplicação pode oferecer uma função Java para consultar o status de um pedido:

```java
public class PedidoTools {

    @Tool("Consulta o status de um pedido pelo identificador")
    public String consultarStatus(Long pedidoId) {
        return "APROVADO";
    }
}
```

O fluxo real é:

1. a pessoa faz uma pergunta;
2. o modelo identifica que uma ferramenta pode ajudar;
3. o modelo solicita a ferramenta e informa os argumentos;
4. a aplicação valida os argumentos;
5. o código Java executa a ação;
6. o resultado volta para o modelo;
7. o modelo produz uma resposta final.

O modelo não executa diretamente o método. Ele apenas indica a intenção de chamada; a aplicação decide se a execução é permitida.

Nunca dê ao modelo acesso irrestrito a banco, arquivos, terminal ou operações destrutivas. Use permissões mínimas, valide entradas e exija confirmação humana para ações sensíveis.

## Agentes

Um **agente (*agent*)** é um fluxo em que o modelo pode escolher etapas e ferramentas para tentar alcançar um objetivo.

Um chatbot que apenas responde texto é relativamente simples. Um agente pode decidir consultar uma API, analisar o resultado, fazer outra consulta e então responder.

Isso aumenta a flexibilidade, mas também aumenta:

- imprevisibilidade;
- custo e quantidade de chamadas;
- dificuldade de testar;
- risco de loop;
- superfície de ataque;
- chance de executar uma ação inadequada.

Comece com um fluxo determinístico quando as etapas forem conhecidas. Use agentes quando a escolha dinâmica realmente trouxer benefício e imponha limites de passos, tempo, custo e ferramentas.

## Estrutura de dependências com Maven

As integrações do LangChain4j são distribuídas em módulos. Um projeto costuma adicionar o módulo principal e o módulo do provedor escolhido:

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>${langchain4j.version}</version>
</dependency>

<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>${langchain4j.version}</version>
</dependency>
```

O número da versão deve ser definido conforme a versão atual compatível com o projeto. Não copie uma versão antiga sem verificar a documentação, as notas de versão e as compatibilidades do provedor.

Em um projeto Quarkus, pode ser preferível usar a extensão oficial do ecossistema Quarkus, pois ela integra configuração, injeção, observabilidade e recursos de desenvolvimento ao modelo de programação do framework.

## LangChain4j e Quarkus

Com a integração de [[Quarkus]], um serviço de IA pode ser registrado e injetado na aplicação. A estrutura pode ficar parecida com:

```java
@RegisterAiService
public interface Assistente {
    String responder(String pergunta);
}
```

Depois, o serviço pode ser injetado em uma classe do [[Backend]]. A configuração do provedor deve ficar fora do código-fonte, normalmente em propriedades, variáveis de ambiente ou secrets do ambiente.

As integrações de Quarkus também podem ajudar com compilação nativa, métricas, tracing, auditoria e uma interface de desenvolvimento. Isso não significa que a aplicação esteja automaticamente segura ou observável: a equipe ainda precisa configurar e revisar esses recursos.

## Segurança

Aplicações com IA lidam frequentemente com textos externos e podem acessar dados importantes. Considere pelo menos:

- proteja as chaves de API e nunca as versione no [[Git]];
- use autenticação e autorização antes de chamar o serviço de IA;
- aplique limites de tamanho, tempo, custo e quantidade de chamadas;
- trate conteúdo do usuário como dado não confiável;
- defenda-se contra prompt injection;
- não coloque segredos no prompt ou no contexto recuperado;
- filtre documentos conforme a permissão da pessoa antes do RAG;
- não permita que uma ferramenta altere dados sem validação;
- registre métricas e erros sem gravar mensagens sensíveis;
- revise o conteúdo enviado a provedores externos;
- defina política de retenção para prompts, respostas e histórico;
- valide a saída antes de convertê-la em comando ou ação;
- mantenha dependências e modelos atualizados.

O LangChain4j ajuda a integrar componentes, mas não substitui as práticas de [[Segurança]] da aplicação.

## Respostas estruturadas

Quando a resposta será usada pelo programa, texto livre pode ser frágil. Prefira solicitar e validar uma estrutura, como [[JSON]], e trate a resposta como entrada externa.

```json
{
  "categoria": "duvida",
  "prioridade": "media",
  "confianca": 0.82
}
```

Mesmo com uma classe Java ou um schema, a resposta precisa ser validada em tempo de execução. O modelo pode produzir campo ausente, valor inválido ou conteúdo que não corresponde ao pedido.

## Testes e observabilidade

Respostas de modelos podem variar. Por isso, teste mais do que uma frase exata:

- formato da resposta;
- presença de campos obrigatórios;
- comportamento quando não há contexto;
- recusa de pedidos proibidos;
- seleção correta de tools;
- permissões aplicadas;
- tempo e custo aproximados;
- comportamento em caso de timeout ou erro do provedor.

Use [[Testes]] com exemplos fixos, avaliações de qualidade e testes de integração quando necessário. Não envie dados reais sensíveis para testes sem autorização.

Também monitore o modelo, o prompt, o número de tokens, o tempo de resposta, falhas, chamadas de tools e documentos recuperados. Ative logs de requisições apenas com cuidado, pois eles podem conter informações pessoais ou segredos.

## LangChain4j não substitui a aplicação

Ele é uma biblioteca dentro da aplicação. Não substitui:

- um modelo de linguagem;
- um [[Backend]] com regras de negócio;
- um [[ORM]] ou banco de dados;
- uma fila ou broker;
- autenticação e autorização;
- observabilidade e testes;
- uma política de privacidade.

Por exemplo, LangChain4j pode ajudar o backend a responder perguntas sobre documentos, mas o backend ainda precisa decidir quem pode consultar cada documento e quais dados podem aparecer na resposta.

## Resumo

LangChain4j é uma biblioteca Java para construir aplicações com modelos de linguagem. Ela oferece integrações com modelos, AI Services, memória, embeddings, RAG, tools e agentes, além de integração com [[Quarkus]]. É uma camada de desenvolvimento, não uma garantia de respostas corretas ou segurança automática. Para usá-la bem, combine prompts claros, validação, permissões, limites, testes e observabilidade.

### Veja também

- [[Java]]
- [[Maven]]
- [[Quarkus]]
- [[Backend]]
- [[Requisição]]
- [[JSON]]
- [[PostgreSQL]]
- [[Redis]]
- [[ORM]]
- [[Git]]
- [[Segurança]]
- [[Testes]]

### Referências oficiais

- [Documentação do LangChain4j](https://docs.langchain4j.dev/)
- [Como começar](https://docs.langchain4j.dev/get-started/)
- [Integração com Quarkus](https://docs.langchain4j.dev/integrations/frameworks/quarkus/)
- [AI Services](https://docs.langchain4j.dev/tutorials/ai-services/)
- [Tools e function calling](https://docs.langchain4j.dev/tutorials/tools/)
- [RAG](https://docs.langchain4j.dev/tutorials/rag/)
