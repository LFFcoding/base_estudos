# O que é RAG?

**RAG** significa *Retrieval-Augmented Generation*, ou **Geração Aumentada por Recuperação**.

É uma técnica que busca informações relevantes em uma fonte de dados e coloca esses trechos no contexto enviado a um modelo de linguagem antes de pedir a resposta.

Uma analogia é um aluno fazendo uma prova com consulta: o modelo é o aluno, os documentos são os livros e o mecanismo de recuperação encontra as páginas mais úteis para a pergunta. O aluno ainda pode interpretar errado, mas passa a consultar uma fonte específica em vez de depender somente do que lembra.

## Por que usar RAG?

Um modelo de linguagem possui conhecimento aprendido durante seu treinamento, mas normalmente não conhece automaticamente:

- documentos internos da empresa;
- regras particulares de um produto;
- dados atualizados de um banco;
- manuais que foram publicados depois do treinamento;
- informações privadas de cada cliente.

Com RAG, esses dados podem ser indexados e recuperados no momento da pergunta. O modelo recebe os trechos encontrados e tenta construir uma resposta baseada neles.

O RAG pode diminuir respostas inventadas (*hallucinations*) quando a recuperação é boa e o modelo é orientado a usar o contexto. Porém, ele não garante que a resposta seja verdadeira.

## Fluxo básico

```text
pergunta do usuário
        ↓
buscar informações relevantes
        ↓
montar prompt com a pergunta e os trechos encontrados
        ↓
enviar ao modelo de linguagem
        ↓
resposta baseada no contexto recuperado
```

O processo costuma ser dividido em duas fases: **indexação** e **recuperação**.

## Fase 1: indexação

A indexação (*indexing* ou *ingestion*) prepara os documentos para serem pesquisados depois.

### 1. Carregar os documentos

Os documentos podem vir de:

- arquivos de texto, PDF ou planilhas;
- páginas internas;
- uma API;
- um [[PostgreSQL]] ou outro banco;
- um sistema de documentos;
- mensagens ou registros autorizados.

A aplicação deve verificar a origem, a versão, a data e as permissões de cada documento.

### 2. Limpar e dividir o conteúdo

Um documento grande é dividido em partes menores chamadas **chunks**. Cada chunk deve conter uma unidade de sentido suficiente para ser entendido quando for recuperado.

```text
manual completo
        ↓
seção 1 | seção 2 | seção 3 | seção 4
```

Chunks muito grandes podem ultrapassar o limite de contexto e trazer informação desnecessária. Chunks muito pequenos podem perder o contexto da explicação.

Não existe um tamanho perfeito para todos os documentos. O tamanho, a sobreposição (*overlap*) e a estratégia de divisão devem ser avaliados com exemplos reais.

### 3. Adicionar metadados

Metadados são informações sobre o trecho, como:

```json
{
  "documento": "manual-produto",
  "secao": "cancelamento",
  "versao": "2026-08",
  "clienteId": "empresa-42",
  "permissao": "suporte"
}
```

Metadados ajudam a filtrar resultados, mostrar fontes e impedir que um usuário receba um documento que não pode consultar.

### 4. Gerar embeddings

Um **embedding** é um vetor numérico que representa características semânticas de um texto. Um modelo de embeddings transforma cada chunk em um vetor.

```text
"Como cancelar um pedido?"
                 ↓
       [0.12, -0.41, 0.77, ...]
```

Textos com significado parecido tendem a ficar próximos no espaço vetorial. A busca pode então encontrar um trecho sobre “cancelamento de compra” mesmo que a pergunta use palavras diferentes.

O modelo de embeddings usado para indexar os documentos deve ser compatível com o usado para transformar a pergunta em vetor.

### 5. Armazenar os vetores

O vetor e o texto original, ou um identificador para encontrá-lo, são guardados em um **embedding store** ou banco vetorial (*vector database*).

Também é possível combinar busca vetorial com busca tradicional por palavras. A busca híbrida pode ser útil quando nomes, códigos, números de contrato ou termos exatos são importantes.

## Fase 2: recuperação

Quando uma pessoa faz uma pergunta, a aplicação executa a segunda fase:

1. recebe e valida a pergunta;
2. aplica autenticação e autorização;
3. transforma a pergunta em embedding, quando usar busca vetorial;
4. procura os chunks mais relevantes;
5. filtra os resultados por metadados e permissões;
6. opcionalmente reordena os resultados com um *reranker*;
7. monta o prompt com pergunta e contexto;
8. envia o prompt ao modelo;
9. valida e apresenta a resposta.

```text
pergunta -> embedding -> busca vetorial -> filtros -> reranking
                                                   ↓
                                  pergunta + contexto -> modelo
```

A qualidade da resposta depende muito da recuperação. Um modelo excelente pode responder mal se receber os trechos errados.

## Componentes importantes

| Componente | Função |
|---|---|
| **Documento** | Conteúdo original que será consultado. |
| **Chunk** | Parte menor do documento. |
| **Metadado** | Informação que descreve o chunk e permite filtros. |
| **Embedding model** | Converte textos em vetores. |
| **Vector store** | Armazena vetores e permite buscar semelhantes. |
| **Retriever** | Executa a busca e devolve trechos relevantes. |
| **Reranker** | Reordena resultados para melhorar a relevância. |
| **Prompt** | Une instruções, pergunta e contexto. |
| **LLM** | Gera a resposta final. |

## Exemplo com LangChain4j

O [[LangChain4j]] oferece componentes para indexar documentos, gerar embeddings, consultar um embedding store e acrescentar o conteúdo recuperado a um AI Service.

Um fluxo conceitual em Java pode ser parecido com este:

```java
EmbeddingStore<TextSegment> store = ...;
EmbeddingModel embeddingModel = ...;

EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
    .embeddingModel(embeddingModel)
    .embeddingStore(store)
    .build();

ingestor.ingest(documento);

ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
    .embeddingStore(store)
    .embeddingModel(embeddingModel)
    .maxResults(3)
    .minScore(0.70)
    .build();
```

Depois, o retriever pode ser ligado a um serviço de IA:

```java
Assistente assistente = AiServices.builder(Assistente.class)
    .chatModel(chatModel)
    .contentRetriever(retriever)
    .build();
```

Os tipos e configurações exatos podem mudar conforme a versão e a integração escolhida. O importante é entender o fluxo: indexar, buscar e acrescentar contexto antes da geração.

Em uma aplicação com [[Quarkus]], o RAG pode ficar em um serviço do [[Backend]] e ser chamado por uma [[Requisição]] autenticada.

## RAG não é fine-tuning

RAG e *fine-tuning* resolvem problemas diferentes:

| RAG | Fine-tuning |
|---|---|
| Busca dados no momento da pergunta. | Ajusta os parâmetros do modelo com exemplos de treinamento. |
| Facilita atualizar documentos sem treinar o modelo novamente. | Pode ensinar estilo, formato ou comportamento especializado. |
| Pode indicar quais trechos foram usados. | Não cria automaticamente uma fonte consultável ou atualizável. |
| Depende da qualidade da recuperação. | Depende da qualidade e segurança dos dados de treinamento. |

Os dois podem ser combinados. Nem todo problema precisa de fine-tuning; muitas aplicações começam com RAG bem configurado.

## RAG não é apenas enviar todos os documentos

Colocar todos os documentos dentro do prompt pode:

- ultrapassar o limite de contexto;
- aumentar o custo;
- aumentar a latência;
- incluir informações irrelevantes;
- expor dados que não deveriam ser vistos.

RAG tenta selecionar apenas o contexto mais relevante. Mesmo assim, envie somente o necessário e aplique os filtros antes de montar o prompt.

## Segurança e autorização

Em RAG, a autorização precisa acontecer antes ou durante a recuperação. Não basta pedir ao modelo para “ignorar” documentos privados.

Um fluxo seguro pode filtrar por `clienteId`, `usuarioId`, grupo e nível de acesso:

```text
usuário autenticado
        ↓
identificar permissões
        ↓
buscar somente chunks permitidos
        ↓
montar contexto
        ↓
enviar ao modelo
```

Boas práticas:

- não indexe documentos sem autorização;
- mantenha metadados de posse e permissão;
- aplique filtros no retriever ou na consulta ao banco;
- não confie em instruções escritas dentro do documento;
- trate documentos como conteúdo não confiável;
- proteja contra prompt injection;
- não grave tokens, senhas ou dados pessoais desnecessários;
- restrinja quem pode inserir, alterar ou excluir documentos;
- versione e audite mudanças na base de conhecimento;
- mantenha os mesmos controles de acesso usados no [[Backend]].

O RAG não substitui as práticas de [[Segurança]] nem a autorização da aplicação.

## Qualidade da recuperação

Uma resposta ruim pode acontecer por vários motivos:

- o documento correto não foi indexado;
- a divisão em chunks separou pergunta e resposta;
- o embedding não representa bem o idioma ou domínio;
- o resultado relevante ficou abaixo do limite de resultados;
- o filtro de relevância ficou permissivo demais;
- o documento está desatualizado;
- o retriever trouxe trechos parecidos, mas incorretos;
- o modelo recebeu contexto demais;
- a pergunta precisava de reescrita ou expansão;
- a informação estava em uma tabela e foi extraída de forma ruim.

Adicionar mais documentos não corrige automaticamente uma recuperação ruim. Ajuste a ingestão, os metadados, o modelo, a consulta, o limite de resultados e o prompt com base em avaliações.

## Como avaliar um RAG?

Crie perguntas reais e uma resposta esperada ou uma fonte esperada. Avalie separadamente:

### Recuperação

- o chunk correto apareceu?
- apareceu dentro dos primeiros resultados?
- os resultados respeitaram as permissões?
- a fonte é atual e adequada?

### Geração

- a resposta usa o contexto recuperado?
- inventa informação ausente?
- responde claramente quando não sabe?
- apresenta fonte ou trecho de apoio?
- segue o formato e o idioma esperados?

### Operação

- qual é a latência?
- qual é o custo por pergunta?
- quantos tokens são usados?
- o que acontece quando o banco ou modelo falha?
- o índice consegue ser atualizado e restaurado?

Use [[Testes]] automatizados para casos de formato, autorização, ausência de contexto e falhas. Para a qualidade semântica, use conjuntos de avaliação revisados por pessoas e métricas adequadas ao domínio.

## Atualização dos documentos

Uma base de RAG precisa acompanhar a fonte original:

1. detectar documento novo ou alterado;
2. remover ou marcar a versão anterior;
3. dividir e gerar novos embeddings;
4. salvar os novos chunks e metadados;
5. validar que o índice foi atualizado;
6. registrar a versão usada.

Não deixe versões antigas e novas misturadas sem uma regra. Uma resposta baseada em um manual obsoleto pode ser pior que uma resposta dizendo que não encontrou informação.

## RAG com PostgreSQL

O [[PostgreSQL]] pode participar de uma solução RAG usando uma extensão ou uma integração que armazene vetores, como a extensão pgvector. Nesse caso, a equipe pode manter dados relacionais, metadados e vetores próximos, mas ainda precisa analisar índices, consultas, volume e permissões.

Uma estrutura simplificada poderia ter:

```text
documentos
- id
- titulo
- versao
- cliente_id

chunks
- id
- documento_id
- texto
- metadados
- embedding
```

O schema real depende do caso. Não coloque um embedding no banco sem planejar o modelo de busca, o índice e a atualização dos documentos.

## Boas práticas

1. **Comece pequeno.** Use poucos documentos e perguntas reais antes de indexar toda a empresa.
2. **Defina a origem da verdade.** Saiba qual sistema mantém o documento oficial.
3. **Guarde metadados.** Eles ajudam em filtros, auditoria, versões e citações.
4. **Teste a divisão.** Verifique se cada chunk pode ser entendido sem o documento inteiro.
5. **Use um limite de relevância.** Não envie qualquer resultado apenas porque ele é o mais próximo disponível.
6. **Controle a quantidade de contexto.** Mais texto não significa necessariamente melhor resposta.
7. **Combine buscas quando necessário.** Termos exatos, códigos e significado semântico têm necessidades diferentes.
8. **Filtre por permissão.** Nunca dependa do modelo para esconder dados privados.
9. **Responda com honestidade.** Oriente o sistema a dizer quando o contexto não basta.
10. **Mostre fontes quando possível.** Isso ajuda a pessoa a conferir a resposta.
11. **Monitore custo e latência.** Indexação e consultas podem chamar modelos várias vezes.
12. **Faça backup do índice e dos documentos.** Embeddings podem ser recriados, mas o processo pode ser caro e demorado.
13. **Versione a configuração.** Registre modelo, chunking, filtros e parâmetros usados.
14. **Proteja dados sensíveis.** Use as orientações de [[Segurança]].

## Resumo

RAG é uma técnica que recupera informações relevantes e as coloca no contexto de um modelo de linguagem antes da geração da resposta. A indexação prepara documentos; a recuperação encontra os trechos adequados. A qualidade depende dos documentos, chunks, embeddings, filtros, permissões e avaliação. RAG ajuda a trabalhar com dados próprios e atualizados, mas não elimina alucinações, autorização, testes ou revisão humana.

### Veja também

- [[LangChain4j]]
- [[Java]]
- [[Maven]]
- [[Quarkus]]
- [[Backend]]
- [[Requisição]]
- [[PostgreSQL]]
- [[ORM]]
- [[Redis]]
- [[JSON]]
- [[Segurança]]
- [[Testes]]

### Referências

- [RAG na documentação do LangChain4j](https://docs.langchain4j.dev/tutorials/rag/)
- [Embedding stores no LangChain4j](https://docs.langchain4j.dev/tutorials/embedding-stores/)
- [Artigo original: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)
