# NoSQL

**NoSQL** é um termo usado para bancos de dados que não dependem principalmente do modelo relacional e das tabelas SQL tradicionais. O nome não significa necessariamente “sem SQL”: significa que existem formas de modelar e consultar dados além do modelo relacional.

NoSQL é uma família de abordagens, não um único banco. Cada produto oferece modelos, consultas, garantias de consistência e formas de escalabilidade diferentes.

## Uma analogia

Um banco relacional é como um arquivo organizado em tabelas, com formulários e relacionamentos bem definidos.

Um banco NoSQL pode ser como:

- uma ficha completa de cada cliente em um documento;
- um dicionário que encontra um valor diretamente pela chave;
- um conjunto de colunas distribuído entre muitos armários;
- um mapa que guarda pessoas e as conexões entre elas.

O formato escolhido depende do tipo de pergunta que a aplicação precisa responder.

## Principais modelos

### Chave-valor (*key-value*)

Cada valor é encontrado por uma chave única:

```text
usuario:42 -> {"nome":"Ana","ativo":true}
```

É adequado para acessos diretos, sessões, preferências e alguns caches. O [[Redis]] é um armazenamento de estruturas de dados em memória que pode ser usado nesse estilo.

### Documento (*document database*)

Os dados são guardados como documentos, frequentemente parecidos com [[JSON]]:

```json
{
  "id": 42,
  "nome": "Ana",
  "enderecos": [
    {"cidade": "São Paulo", "principal": true}
  ]
}
```

Documentos podem ter campos diferentes, mas flexibilidade não elimina a necessidade de validar o formato.

O [[MongoDB]] é um exemplo conhecido de banco orientado a documentos. Ele guarda documentos BSON em coleções e permite consultar campos, listas e documentos aninhados.

### Colunas largas (*wide-column*)

Os dados são organizados para distribuir grandes volumes e consultar determinados grupos de colunas. Esse modelo aparece em sistemas que precisam de escala horizontal e alto volume de escrita.

### Grafos (*graph database*)

O foco são entidades e relações entre elas. É útil para redes sociais, recomendações, mapas de dependência e caminhos entre objetos.

### Séries temporais (*time series*)

O foco são valores associados a tempo, como métricas, sensores, preços e eventos. O armazenamento pode ser otimizado para inserir e consultar intervalos temporais.

## NoSQL não significa ausência de estrutura

Um banco NoSQL pode não exigir o mesmo esquema fixo de um banco relacional, mas a aplicação ainda possui um formato esperado.

Se um documento deveria conter `usuarioId` e `status`, a equipe precisa decidir:

- quais campos são obrigatórios;
- quais tipos são permitidos;
- como os documentos antigos serão atualizados;
- como os dados serão consultados;
- como a aplicação lidará com documentos inválidos.

Essa estrutura pode ser aplicada pela aplicação, pelo banco, por validações ou por contratos de dados.

## NoSQL e banco relacional

| Característica | Relacional | NoSQL |
| --- | --- | --- |
| Organização | tabelas, linhas e colunas | documentos, chaves, colunas ou grafos |
| Relacionamentos | fortes e consultados com joins | frequentemente modelados dentro do documento ou por referências |
| Esquema | geralmente definido antes | pode ser mais flexível, dependendo do produto |
| Consultas | SQL e operações relacionais | linguagem ou API específica do banco |
| Escala | vertical e também horizontal, conforme a arquitetura | frequentemente planejada para distribuição horizontal |
| Consistência | transações e garantias relacionais maduras | varia bastante entre produtos e operações |

O [[PostgreSQL]] é relacional e adequado para dados principais, relações, transações e consultas complexas. Um banco NoSQL pode ser adequado quando o padrão de acesso, o volume ou a necessidade de distribuição justificarem essa escolha.

## Como escolher

Não escolha NoSQL apenas porque ele parece mais rápido ou moderno. Comece pelas perguntas que o sistema precisa responder:

1. quais dados serão lidos juntos?
2. quais buscas precisam ser rápidas?
3. qual volume de leitura e escrita é esperado?
4. quais relações precisam ser consultadas?
5. quais operações precisam ser atômicas?
6. que nível de consistência é necessário?
7. como os dados serão corrigidos, auditados e excluídos?
8. a equipe conhece e consegue operar o banco escolhido?

No NoSQL, é comum modelar os dados a partir das consultas. Isso pode significar duplicar algumas informações para evitar joins, mas exige uma estratégia clara para atualizar as cópias.

## Exemplo de decisão

Uma aplicação pode usar:

- [[PostgreSQL]] para usuários, pedidos, pagamentos e relacionamentos importantes;
- [[Redis]] para sessões, [[Cache|cache]] e contadores temporários;
- um banco de documentos para catálogos cujo formato muda com frequência;
- [[MongoDB]] para dados que se encaixam bem no modelo de documentos;
- uma [[Filas|fila]] para trabalhos assíncronos.

Usar mais de uma tecnologia aumenta as capacidades, mas também aumenta a complexidade operacional. Cada banco precisa de monitoramento, backups, segurança, atualização e conhecimento da equipe.

## NoSQL na AWS

Na AWS, o [Amazon DynamoDB](https://aws.amazon.com/documentation-overview/dynamodb/) é um banco NoSQL gerenciado que oferece modelos chave-valor e documento, com foco em baixa latência e escala horizontal.

O DynamoDB reduz a administração de servidores, mas exige modelar as chaves de partição, índices, limites de leitura e escrita, custos e padrões de acesso antes de construir as consultas. Ele não é uma cópia do PostgreSQL e não deve ser escolhido sem entender suas garantias e seu modelo de dados.

Em uma [[VPS]], a equipe pode instalar e operar uma solução NoSQL compatível, mas assume a responsabilidade por armazenamento, cluster, atualizações, backups, segurança e recuperação. O serviço gerenciado da AWS reduz parte desse trabalho em troca de custo, regras da plataforma e dependência do provedor.

## Segurança e consistência

- proteja o banco com rede privada, autenticação e permissões mínimas;
- não confie na flexibilidade do esquema para aceitar qualquer entrada;
- valide documentos no [[Backend]];
- não coloque dados pessoais em chaves ou logs sem necessidade;
- defina quem pode ler, alterar e excluir cada coleção ou partição;
- entenda se uma leitura pode retornar dados antigos;
- use transações ou operações atômicas quando o caso exigir;
- planeje retenção, criptografia, backups e restauração;
- monitore consultas lentas, partições quentes, memória e limites de capacidade.

NoSQL não elimina os problemas de segurança ou consistência. Ele apenas oferece modelos diferentes para resolvê-los.

## Boas práticas

- modele primeiro os padrões de acesso importantes;
- documente o formato dos documentos e as versões existentes;
- evite documentos grandes demais;
- não duplique dados sem um plano de atualização;
- crie índices somente quando houver uma consulta que os justifique;
- teste volume, concorrência, falhas e restauração;
- compare o custo total, incluindo operação e treinamento da equipe;
- prefira o [[PostgreSQL]] quando o problema for naturalmente relacional e ele atender aos requisitos;
- use NoSQL quando o modelo, a escala ou o padrão de acesso trouxerem uma vantagem real.

**NoSQL é uma família de bancos com modelos alternativos ao relacional. A escolha correta depende das consultas, da consistência, da escala e da capacidade de operar a solução.**
