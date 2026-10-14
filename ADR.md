# O que é ADR?

**ADR** significa **Architecture Decision Record** (*registro de decisão arquitetural*). É um documento curto que registra uma decisão importante sobre a estrutura, a tecnologia ou o funcionamento de um sistema.

Uma decisão arquitetural pode ser, por exemplo:

- escolher [[PostgreSQL]] em vez de outro [[Banco de dados|banco de dados]];
- organizar o [[Backend]] como um monólito modular;
- usar [[Docker Compose]] para executar serviços localmente;
- colocar o [[Caddy]] na frente da aplicação como proxy reverso;
- escolher [[Next.js]] e [[React]] para o [[Frontend]];
- adotar uma estratégia de autenticação descrita em [[Segurança]].

O ADR registra não apenas **o que foi escolhido**, mas também **por que a escolha foi feita**, quais alternativas foram consideradas e quais consequências são esperadas.

Uma analogia: imagine uma família decidindo morar em uma determinada cidade. O ADR seria o registro que explica o motivo da escolha, as outras cidades avaliadas, os benefícios esperados e as dificuldades aceitas. No futuro, outra pessoa consegue entender a decisão sem precisar adivinhar o que aconteceu em uma conversa antiga.

## Por que um ADR é útil?

Projetos acumulam decisões. Depois de alguns meses, pode ser difícil lembrar:

- por que uma tecnologia foi escolhida;
- qual problema a arquitetura deveria resolver;
- quais alternativas foram descartadas;
- quais limitações eram conhecidas;
- quem precisa ser consultado antes de mudar a decisão.

Sem um registro, a equipe pode repetir discussões, desfazer uma escolha sem perceber suas consequências ou concluir que uma decisão foi feita por acaso.

O ADR transforma uma decisão importante em conhecimento compartilhado. Ele é especialmente útil para decisões que afetam várias partes, como [[Backend]], [[Frontend]], [[Banco de dados|banco de dados]], infraestrutura, deploy, [[Segurança]] e [[Testes]].

## O que deve ser registrado?

Um ADR geralmente contém:

1. **Identificador:** número ou código que diferencia o documento;
2. **Título:** frase curta com a decisão principal;
3. **Status:** situação atual da decisão;
4. **Data:** quando a decisão foi registrada;
5. **Contexto:** problema, necessidade ou restrição;
6. **Decisão:** escolha feita pela equipe;
7. **Alternativas:** opções consideradas e motivos para não escolhê-las;
8. **Consequências:** benefícios, custos, riscos e limitações;
9. **Próximos passos:** ações necessárias para colocar a decisão em prática.

Nem todo ADR precisa ter muitas páginas. O objetivo é preservar o raciocínio importante com clareza, não criar uma documentação difícil de manter.

## Estrutura simples de um ADR

Um modelo prático pode ser escrito em Markdown:

```markdown
# ADR-0001: Título da decisão

- Status: Proposta
- Data: 2026-10-06

## Contexto

Qual problema precisamos resolver? Quais restrições existem?

## Decisão

Qual opção foi escolhida?

## Alternativas consideradas

- Opção A: por que foi considerada e por que não foi escolhida;
- Opção B: por que foi considerada e por que não foi escolhida.

## Consequências

### Benefícios

- resultado positivo esperado;

### Custos e riscos

- esforço, limitação ou risco aceito.

## Próximos passos

- ação necessária para aplicar a decisão.
```

O formato pode mudar conforme o projeto. O mais importante é que outra pessoa consiga entender o problema, a escolha e seus efeitos.

## Status de uma decisão

O status mostra se a decisão ainda está sendo discutida ou se já orienta o projeto. Alguns estados comuns são:

- **Proposta (*proposed*):** a decisão foi sugerida, mas ainda não foi aprovada;
- **Aceita (*accepted*):** a equipe aprovou a decisão e deve segui-la;
- **Rejeitada (*rejected*):** a proposta foi analisada, mas não será usada;
- **Obsoleta (*deprecated*):** a decisão não deve mais orientar mudanças novas;
- **Substituída (*superseded*):** outra decisão tomou o seu lugar.

Uma decisão aceita não precisa ser perfeita para sempre. Ela representa a melhor escolha conhecida dentro do contexto e das restrições existentes naquele momento.

## Exemplo: escolher PostgreSQL

O exemplo abaixo mostra como registrar uma decisão. Ele é ilustrativo; uma decisão real deve considerar os requisitos concretos do projeto.

```markdown
# ADR-0001: Usar PostgreSQL como banco principal

- Status: Aceita
- Data: 2026-10-06

## Contexto

O sistema precisa armazenar dados relacionais, aplicar transações e consultar
informações com segurança. A stack de estudos usa Java, Quarkus e Maven no
backend, e a aplicação será executada em containers.

## Decisão

Usaremos PostgreSQL como banco de dados relacional principal.

## Alternativas consideradas

- MySQL: atende ao caso, mas a equipe quer concentrar o estudo em PostgreSQL;
- MongoDB: é uma alternativa NoSQL, mas não é a melhor escolha para os
  relacionamentos e transações deste contexto;
- banco gerenciado: pode reduzir a administração, mas não é necessário no
  primeiro ambiente de estudos.

## Consequências

### Benefícios

- suporte a transações e relacionamentos;
- boa integração com ferramentas do backend;
- conhecimento alinhado à stack do projeto.

### Custos e riscos

- será necessário cuidar de backup, migrações e credenciais;
- a equipe precisará aprender a monitorar o banco;
- uma mudança futura para outro banco exigirá adaptação.
```

No exemplo, [[Java]], [[Quarkus]] e [[Maven]] representam tecnologias da stack do backend. [[MongoDB]] é uma alternativa de banco [[NoSQL]] considerada, mas não escolhida para aquele contexto.

Esse registro explica a escolha sem dizer que PostgreSQL é sempre o melhor banco. A resposta depende dos requisitos, do conhecimento da equipe, do custo e do ambiente.

## ADR não é uma especificação completa

Um ADR responde principalmente a estas perguntas:

> Qual decisão foi tomada, em qual contexto e por quê?

Ele não substitui todos os outros documentos do projeto:

- uma especificação descreve o que o sistema deve fazer;
- um desenho de arquitetura mostra componentes e relações;
- uma tarefa descreve o trabalho que precisa ser executado;
- um tutorial ensina como realizar uma atividade;
- um ADR explica uma decisão e seu raciocínio.

Um projeto pode ter todos esses documentos. O ADR funciona como uma placa no caminho: não mostra todos os detalhes da estrada, mas explica por que aquele caminho foi escolhido.

## Como criar um ADR

Um fluxo simples é:

1. perceber que existe uma decisão com impacto relevante;
2. escrever o contexto e as restrições;
3. listar as alternativas razoáveis;
4. comparar benefícios, custos e riscos;
5. registrar a decisão escolhida;
6. revisar o texto com as pessoas afetadas;
7. versionar o arquivo no [[Git]];
8. compartilhar o registro com a equipe pelo [[GitHub]] ou pela ferramenta usada no projeto.

Em decisões simples, a revisão pode ser rápida. Em decisões que afetam segurança, dados ou produção, é importante ouvir as pessoas responsáveis por essas áreas antes de aceitar o ADR.

## ADRs e mudanças na arquitetura

Quando uma decisão aceita deixa de ser adequada, evite apagar o histórico. Há duas formas comuns de lidar com isso:

1. atualizar o status do ADR antigo para **Substituída**;
2. criar um novo ADR explicando a mudança e apontando para o registro anterior.

Exemplo:

```text
ADR-0001: Usar banco PostgreSQL        → Substituída
ADR-0007: Adotar serviço gerenciado    → Aceita
```

O primeiro registro do exemplo se refere ao arquivo [[PostgreSQL]]; o segundo representa uma decisão posterior sobre a forma de administrar o banco.

O novo ADR deve explicar o que mudou no contexto. Talvez o sistema tenha crescido, a equipe tenha adquirido outra necessidade ou o custo de administrar a solução antiga tenha deixado de fazer sentido.

Preservar essa sequência ajuda a entender a evolução da arquitetura. A decisão antiga pode ter sido correta quando foi tomada, mesmo que não seja a melhor escolha atual.

## Como organizar os arquivos

Em muitos projetos, os ADRs recebem um número e um nome curto:

```text
ADR-0001-usar-postgresql.md
ADR-0002-executar-com-docker-compose.md
ADR-0003-usar-caddy-como-proxy.md
```

Neste repositório, os arquivos de estudo ficam na pasta principal. Em um projeto real, a equipe pode escolher uma pasta específica para ADRs, desde que o local e o padrão sejam consistentes.

O número ajuda a ordenar os registros. O nome deve indicar a decisão sem depender apenas de uma data.

## Boas práticas

- registrar o motivo da decisão, não somente a tecnologia escolhida;
- escrever enquanto o contexto ainda está fresco;
- usar linguagem simples e fatos verificáveis;
- listar alternativas reais, e não opções inventadas apenas para preencher o documento;
- registrar custos e riscos junto dos benefícios;
- separar uma decisão grande em decisões menores quando isso facilitar a compreensão;
- não transformar cada preferência pessoal em uma decisão arquitetural;
- manter o ADR versionado com o código e a configuração relacionados;
- revisar decisões antigas quando o contexto mudar;
- criar um novo ADR quando uma decisão aceita for substituída;
- ligar o ADR a issues, pull requests ou documentos relacionados quando a ferramenta permitir;
- evitar copiar segredos, tokens ou dados pessoais para o registro.

Um ADR deve ser curto o suficiente para ser consultado e detalhado o suficiente para preservar o raciocínio. Se ninguém consegue encontrar ou ler o registro, ele não está cumprindo sua função.

## ADRs e segurança

Decisões arquiteturais podem mudar o risco do sistema. Por isso, um ADR que envolve autenticação, autorização, dados pessoais, rede ou infraestrutura deve registrar também as consequências de segurança.

Por exemplo, ao escolher um serviço externo, considere:

- quais dados serão enviados;
- como a comunicação será protegida;
- quem poderá acessar o serviço;
- como as credenciais serão armazenadas;
- o que acontece se o serviço ficar indisponível;
- como a equipe revogará o acesso.

O ADR não deve guardar a senha ou a chave real. Ele pode dizer onde o segredo é administrado e quais controles são necessários. A decisão deve complementar as práticas de [[Segurança]], e não substituí-las.

## ADRs, testes e entrega

Uma decisão arquitetural deve ser validada quando possível. Por exemplo:

- um ADR sobre o [[Backend]] pode ser acompanhado de testes de integração;
- uma escolha de banco pode ser avaliada com dados representativos;
- uma decisão sobre o [[Frontend]] pode ser verificada com testes de componente;
- uma decisão de infraestrutura pode ser exercitada em um ambiente separado;
- um pipeline de [[CI-CD]] pode verificar se a configuração continua válida.

Os [[Testes]] não provam que a decisão é perfeita, mas ajudam a detectar quando a implementação deixou de cumprir os motivos que levaram à escolha.

## Resumo

> **ADR é um registro curto que explica uma decisão importante de arquitetura, o contexto que levou a ela e suas consequências.**

Ele evita que o conhecimento fique apenas na memória das pessoas. Quando é mantido junto do projeto e revisado com o tempo, ajuda a equipe a tomar decisões mais conscientes e a entender como o sistema evoluiu.
