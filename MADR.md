# O que é MADR?

**MADR** significa **Markdown Architectural Decision Records** (*registros de decisões arquiteturais em Markdown*). É uma convenção de modelos para escrever [[ADR|registros de decisões arquiteturais]] em Markdown.

Um ADR é o registro de uma decisão arquitetural; MADR é uma forma específica e estruturada de escrever esse registro. Portanto, MADR não é uma ferramenta, linguagem de programação ou substituto do conceito de ADR: é uma convenção que ajuda a documentar decisões com consistência.

## Para que serve?

MADR ajuda uma equipe a registrar o contexto de uma decisão, as opções consideradas, a escolha feita e seus efeitos esperados. Como os arquivos são Markdown, podem ser versionados junto ao projeto usando [[Git]], revisados em pull requests e consultados sem uma ferramenta especial.

Isso é útil, por exemplo, para explicar por que um projeto escolheu [[PostgreSQL]], adotou uma arquitetura específica para o [[Backend]] ou decidiu executar serviços com [[Docker Compose]]. O registro permite que pessoas futuras compreendam o raciocínio sem depender da memória de quem participou da conversa original.

## Seções comuns do modelo MADR

O modelo completo oficial apresenta seções como:

- **Título:** resume o problema e a solução escolhida;
- **Contexto e declaração do problema:** descreve a situação, o problema e o escopo da decisão;
- **Fatores da decisão:** lista requisitos, restrições e qualidades que influenciam a escolha;
- **Opções consideradas:** registra as alternativas avaliadas;
- **Resultado da decisão:** declara a opção escolhida e justifica o motivo;
- **Consequências:** aponta efeitos positivos e negativos da decisão;
- **Confirmação:** explica como verificar se a implementação segue a decisão;
- **Prós e contras das opções:** compara as alternativas de forma explícita;
- **Mais informações:** reúne referências, evidências e detalhes relacionados.

Nem todas as seções são obrigatórias em todas as variantes do modelo. O modelo completo é mais detalhado; o modelo mínimo mantém apenas as seções essenciais. Também existem versões anotadas, que explicam como preencher cada parte, e versões em branco, prontas para copiar.

## Exemplo simplificado

```markdown
---
status: aceito
date: 2026-10-19
---

# Usar PostgreSQL como banco de dados principal

## Contexto e problema

O sistema precisa armazenar dados relacionados e realizar operações transacionais.

## Fatores da decisão

- suporte a relacionamentos e transações;
- integração com a stack existente;
- facilidade de operação pela equipe.

## Opções consideradas

- PostgreSQL;
- MongoDB;
- MySQL.

## Resultado da decisão

Escolhemos PostgreSQL por atender aos requisitos relacionais e transacionais.

### Consequências

- bom suporte a consultas relacionais e transações;
- será necessário cuidar de migrações, backups e credenciais.

### Confirmação

Revisar o esquema e os testes de integração para confirmar o uso esperado.
```

O exemplo é didático, não uma decisão real deste repositório. Em um projeto, o texto deve registrar as opções que de fato foram consideradas e as razões específicas da equipe. O status e outros metadados podem ser ajustados ao vocabulário adotado pelo projeto.

## Como organizar decisões MADR

O projeto oficial sugere copiar um modelo para uma pasta de decisões, como `docs/decisions`, e criar um arquivo por decisão com um identificador numérico e título, por exemplo:

```text
0001-usar-postgresql.md
0002-executar-com-docker-compose.md
```

A pasta e o padrão de nomes são convenções, não uma exigência técnica. Neste cofre, as notas ficam na pasta principal; cada projeto pode escolher sua própria organização, desde que seja consistente e fácil de encontrar.

## Boas práticas

- registre a decisão quando o contexto ainda estiver claro;
- deixe explícitos o problema, o escopo e os fatores que influenciaram a escolha;
- compare alternativas reais, sem inventar opções apenas para preencher o modelo;
- descreva consequências positivas e negativas;
- escreva de forma concisa, mas suficiente para uma pessoa nova entender o raciocínio;
- mantenha o histórico: se uma decisão aceita mudar, registre a nova decisão e relacione-a à anterior em vez de apagar o motivo original;
- não inclua senhas, tokens, chaves ou outros segredos no documento;
- adapte o modelo ao contexto sem remover informações essenciais para entender a escolha.

MADR é um formato flexível: não é necessário preencher cada seção quando ela não se aplica. A estrutura serve para orientar o raciocínio, não para transformar cada escolha em burocracia.

## Relação com ADR

[[ADR]] explica o conceito geral de registro de decisão arquitetural e apresenta um modelo simples. MADR é uma convenção conhecida para estruturar esses registros em Markdown, com modelos completos ou mínimos e seções como fatores da decisão, confirmação e comparação de opções.

## Referências

- [Repositório oficial do MADR](https://github.com/adr/madr)
- [Modelo MADR completo e anotado](https://github.com/adr/madr/blob/develop/template/adr-template.md)
