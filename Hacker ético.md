# O que é um hacker ético?

Um **hacker ético** (*ethical hacker*) é uma pessoa autorizada a avaliar a segurança de sistemas, redes ou aplicações para encontrar fraquezas antes que alguém as use para causar danos.

Uma analogia: é como contratar alguém para testar portas, janelas e alarmes de uma casa, com autorização do proprietário e regras claras. A pessoa registra o que encontrou e ajuda a corrigir os problemas; não leva objetos, não danifica a casa nem entra em outros imóveis.

O termo **hacker** descreve alguém que explora profundamente como uma tecnologia funciona. A intenção e a autorização distinguem um trabalho ético de uma invasão. Mesmo com boa intenção, testar um sistema sem permissão pode causar prejuízo e consequências legais.

## O que faz?

O trabalho pode incluir:

- revisar configurações e código;
- procurar falhas em aplicações web, APIs, redes ou dispositivos;
- verificar se controles de acesso funcionam como esperado;
- avaliar se dados sensíveis estão protegidos;
- analisar resultados de ferramentas de segurança;
- demonstrar, dentro do escopo combinado, o impacto de uma vulnerabilidade;
- explicar como corrigir e reduzir o risco;
- testar novamente para confirmar que a correção funcionou.

Um hacker ético trabalha junto das equipes responsáveis e ajuda a melhorar a [[Segurança]] do produto. Não basta encontrar uma falha: é preciso descrevê-la de forma compreensível, proteger as evidências e apoiar a correção.

## Autorização e escopo

Antes de qualquer teste, as partes precisam combinar por escrito:

- quem autoriza o trabalho e quem é responsável pelo sistema;
- quais domínios, aplicações, endereços e ambientes podem ser testados;
- quais técnicas e horários são permitidos;
- quais ações são proibidas, como alterar dados reais ou interromper serviços;
- como comunicar uma falha urgente;
- como guardar e apagar dados e evidências;
- quando o teste começa e termina.

O **escopo (*scope*)** é a lista precisa do que pode ser avaliado. Ter acesso a um sistema não significa ter permissão para testar todos os sistemas conectados a ele. Serviços de terceiros, provedores e dados de clientes também podem exigir autorização própria.

Exemplo de escopo resumido:

```text
Permitido:
- aplicação de homologação em https://app-teste.exemplo;
- contas de teste fornecidas pela equipe;
- verificar autenticação, autorização e validação de entradas.

Não permitido:
- ambiente de produção;
- negação de serviço;
- acessar dados de clientes;
- testar domínios de fornecedores.

Janela: 20/10/2026, das 14h às 17h.
Contato para interrupção: responsável técnico indicado no acordo.
```

Este é apenas um exemplo de organização; a autorização real deve ser aprovada por quem tem autoridade sobre os ativos e considerar as regras aplicáveis.

## Etapas de um trabalho autorizado

Um teste de segurança costuma seguir etapas como estas:

1. **Planejar:** entender o objetivo, os riscos e o escopo;
2. **Preparar:** combinar acessos de teste, contatos e plano de interrupção;
3. **Reconhecer:** identificar componentes e funcionalidades permitidos;
4. **Avaliar:** procurar falhas com revisão manual e ferramentas adequadas;
5. **Validar com cuidado:** confirmar o problema sem causar impacto desnecessário;
6. **Documentar:** registrar evidências mínimas, impacto e condições observadas;
7. **Reportar:** comunicar os resultados às pessoas responsáveis;
8. **Corrigir e retestar:** verificar se as mudanças resolveram o problema.

A investigação pode usar análise estática ou dinâmica e, quando necessário, [[Reverse engineering|engenharia reversa]]. O tipo de técnica depende do escopo e da autorização.

## Pentest e varredura de vulnerabilidades

Uma **varredura de vulnerabilidades** (*vulnerability scan*) usa ferramentas para procurar sinais conhecidos de problemas. Ela pode ser rápida e cobrir muitos componentes, mas pode produzir falsos positivos e não entende sempre o contexto do sistema.

Um **teste de invasão** (*penetration test* ou pentest) avalia, dentro de um escopo autorizado, se uma ou mais fraquezas podem ser combinadas para demonstrar um impacto realista. Ele inclui análise humana e requer controles para evitar danos.

| Atividade | Principal objetivo | Limite comum |
| --- | --- | --- |
| Varredura de vulnerabilidades | Encontrar possíveis problemas conhecidos | Pode apontar itens que não são exploráveis naquele contexto |
| Pentest | Avaliar caminhos de ataque e impacto dentro do escopo | Exige planejamento, autorização e cuidado operacional |
| Revisão de código | Identificar problemas diretamente na implementação | Pode não mostrar como a aplicação se comporta em produção |
| Auditoria de configuração | Conferir permissões e opções de serviços | Depende de acesso e critérios definidos |

Essas atividades se complementam. Um resultado automático precisa ser analisado; um pentest não substitui revisões contínuas, [[Testes]] ou boas práticas de desenvolvimento.

## Exemplo em uma aplicação

Imagine uma equipe que quer verificar se pessoas usuárias conseguem acessar somente os registros de sua própria organização.

Em um ambiente de homologação, com contas e dados fictícios, a equipe pode:

1. criar duas organizações de teste;
2. autenticar uma conta de cada organização;
3. verificar se cada conta acessa seus próprios registros;
4. confirmar que a API rejeita o acesso cruzado;
5. registrar a rota, o resultado e o impacto sem copiar dados reais;
6. comunicar a falha e recomendar uma verificação de autorização no [[Backend]].

O ponto é verificar a regra de acesso combinada, sem consultar registros de clientes ou alterar dados de produção. Esconder uma tela no [[Frontend]] não é suficiente: a permissão também precisa ser validada pela API.

## Como escrever um relatório

Um relatório útil normalmente registra:

- **resumo:** o que foi avaliado e qual foi a conclusão geral;
- **escopo:** sistemas e ambientes incluídos;
- **achado:** descrição objetiva da falha;
- **impacto:** o que poderia acontecer se ela fosse explorada;
- **evidência:** demonstração mínima, sem dados sensíveis desnecessários;
- **gravidade:** prioridade baseada em impacto e facilidade de exploração;
- **correção sugerida:** mudança que reduz o risco;
- **limitações:** partes que não foram testadas;
- **reteste:** resultado da verificação após a correção.

Uma boa recomendação é específica. “Melhorar a segurança” não diz o que fazer. “Verificar no servidor se o identificador do registro pertence à organização da sessão antes de devolver o resultado” é mais acionável.

## Boas práticas

- obter autorização explícita antes dos testes;
- manter um escopo escrito e conferir os alvos antes de cada atividade;
- usar ambientes de teste e dados fictícios sempre que possível;
- avisar os responsáveis sobre horários e riscos;
- interromper o teste se surgir impacto inesperado;
- coletar apenas as evidências necessárias;
- proteger relatórios, credenciais e dados encontrados;
- não publicar detalhes de uma falha antes de combinar a divulgação responsável;
- distinguir fatos observados de hipóteses;
- recomendar correções claras e proporcionais;
- confirmar a correção com um reteste;
- registrar o trabalho e as decisões no processo de desenvolvimento.

O [[Git]] pode guardar correções e documentação sem incluir segredos. O [[CI-CD]] pode automatizar verificações repetíveis, mas não substitui a avaliação humana nem a autorização para testar sistemas.

## O que não é trabalho ético

Não é trabalho ético acessar sistemas sem autorização, roubar dados, manter acesso escondido, interromper serviços, alterar registros ou usar uma falha para obter vantagem pessoal. A intenção alegada não substitui a permissão do responsável.

Se uma pessoa encontrar uma falha por acaso, deve evitar continuar explorando, não copiar dados, preservar somente as informações mínimas necessárias e procurar um canal apropriado para comunicar o problema.

## Resumo

> **Hacker ético é um profissional autorizado que avalia a segurança de sistemas, comunica as falhas com responsabilidade e ajuda a corrigi-las.**

Autorização, escopo, cuidado com dados e comunicação clara fazem parte do trabalho tanto quanto o conhecimento técnico.
