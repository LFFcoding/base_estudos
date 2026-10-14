# O que é reverse engineering?

**Reverse engineering** significa **engenharia reversa**. É o processo de estudar um produto, programa, protocolo ou sistema já existente para descobrir como ele funciona.

Na programação tradicional, a pessoa começa com uma ideia e constrói o sistema. Na engenharia reversa, ela começa com o resultado pronto e tenta entender sua estrutura, suas regras e seu comportamento.

Uma analogia é desmontar um relógio para entender como as engrenagens fazem os ponteiros se moverem. O objetivo não é apenas separar as peças, mas descobrir a função de cada uma e como elas trabalham juntas.

Engenharia reversa não significa automaticamente invadir, roubar código ou quebrar uma proteção. Ela pode ser uma atividade legítima de manutenção, integração, segurança, investigação e aprendizado, desde que seja feita com autorização e respeite as regras aplicáveis.

## Para que serve?

Alguns objetivos comuns são:

- entender um sistema antigo que não possui documentação suficiente;
- integrar uma aplicação com um equipamento ou protocolo desconhecido;
- investigar um erro difícil de reproduzir;
- verificar a segurança de um programa autorizado;
- analisar um arquivo ou formato de dados;
- migrar informações para um sistema novo;
- verificar a compatibilidade entre produtos;
- estudar como uma tecnologia funciona;
- analisar um programa malicioso em um laboratório isolado;
- descobrir dependências e comportamentos escondidos.

Em uma aplicação de [[Agtech]], por exemplo, a engenharia reversa pode ajudar a entender o protocolo de um sensor antigo antes de conectá-lo a um novo [[Backend]]. Isso não dispensa a autorização do proprietário do equipamento nem a validação dos dados recebidos.

## Engenharia reversa não é apenas para código

Ela pode ser aplicada a:

- **software:** aplicações, bibliotecas, executáveis e bytecode;
- **protocolos:** formatos de mensagens, autenticação e comunicação entre sistemas;
- **arquivos:** documentos, imagens, bases de dados e formatos proprietários;
- **hardware:** placas, sensores, máquinas e componentes eletrônicos;
- **sistemas distribuídos:** serviços, filas, APIs e integrações;
- **malware:** programas maliciosos analisados para defesa e resposta a incidentes.

O tipo de evidência disponível muda conforme o caso. Um programa pode fornecer um executável, enquanto um equipamento pode fornecer sinais elétricos, mensagens ou respostas a comandos.

## O que pode ser descoberto?

Durante a análise, a pessoa pode tentar descobrir:

- quais entradas o sistema aceita;
- quais saídas ele produz;
- quais estados existem;
- como os dados são transformados;
- quais serviços ou bibliotecas são chamados;
- como ocorre a autenticação;
- quais erros são retornados;
- quais regras de negócio parecem existir;
- como os componentes se comunicam;
- quais limitações e dependências o sistema possui.

O resultado não precisa ser o código-fonte original. Muitas vezes, o produto da engenharia reversa é um modelo de comportamento, um diagrama, uma especificação de protocolo ou uma documentação para integração.

## Tipos de acesso

### Caixa-preta (*black box*)

Na análise de caixa-preta, a pessoa não vê o código interno. Ela observa entradas, saídas e efeitos externos.

Por exemplo, pode enviar uma [[Requisição]] para uma API autorizada e observar o status HTTP, os cabeçalhos e o [[JSON]] retornado. É parecido com testar uma máquina apenas usando seus botões e observando o resultado.

### Caixa-branca (*white box*)

Na análise de caixa-branca, o código-fonte, os projetos ou os símbolos internos estão disponíveis. A pessoa consegue acompanhar as classes, os métodos, as dependências e o fluxo de execução.

Esse tipo de acesso costuma produzir conclusões mais precisas, mas ainda exige entendimento do ambiente, das configurações e dos dados usados em execução.

### Caixa-cinza (*gray box*)

Na análise de caixa-cinza, existe acesso parcial. A pessoa conhece algumas regras, possui documentação incompleta ou consegue observar parte da implementação.

Auditorias autorizadas e integrações com sistemas de terceiros frequentemente usam esse modelo.

## Análise estática

**Análise estática (*static analysis*)** é o estudo de um artefato sem executá-lo. O artefato pode ser código-fonte, bytecode, um executável, um arquivo ou uma configuração.

Ela pode revelar:

- textos e mensagens de erro;
- nomes de classes e funções;
- bibliotecas usadas;
- chamadas para outros serviços;
- formatos de dados;
- constantes e configurações;
- permissões solicitadas;
- possíveis caminhos de execução.

Exemplos de ferramentas:

- `file`, para identificar o tipo de um arquivo;
- `strings`, para procurar textos legíveis;
- `javap`, para observar informações e bytecode de classes Java;
- descompiladores, para produzir uma aproximação de código legível;
- analisadores de dependência e de configuração.

Um comando simples para observar uma classe compilada em [[Java]] é:

```bash
javap -c MinhaClasse.class
```

O comando pode mostrar instruções de bytecode e a estrutura pública da classe. Ele não recupera necessariamente o código-fonte original, os nomes removidos durante a compilação ou os comentários.

## Análise dinâmica

**Análise dinâmica (*dynamic analysis*)** é o estudo do sistema enquanto ele está sendo executado em um ambiente controlado.

Ela pode observar:

- chamadas de função;
- arquivos lidos e gravados;
- conexões de rede;
- mensagens trocadas;
- consumo de memória e CPU;
- alterações de estado;
- exceções e códigos de erro.

Ferramentas de depuração (*debuggers*), instrumentação, logs e captura de tráfego podem ajudar. Em um sistema com [[TLS]], a comunicação deve ser analisada somente em ambientes e conexões para os quais exista autorização.

A análise dinâmica revela comportamentos que não aparecem no arquivo parado, mas pode deixar passar caminhos que não foram executados durante o experimento.

## Descompilação, desmontagem e depuração

Esses termos são relacionados, mas não significam exatamente a mesma coisa:

- **descompilação (*decompilation*):** tenta transformar código compilado em uma representação parecida com código-fonte;
- **desmontagem (*disassembly*):** transforma instruções de máquina em assembly;
- **depuração (*debugging*):** acompanha a execução para observar valores, chamadas e estados;
- **instrumentação (*instrumentation*):** adiciona observações ou medições durante a execução;
- **análise de fluxo:** identifica como dados e decisões percorrem o sistema.

O resultado dessas ferramentas é uma pista, não uma cópia perfeita do projeto original. Compiladores podem otimizar código, remover nomes e reorganizar instruções.

## Um processo de engenharia reversa

Um processo responsável pode seguir estas etapas:

1. **definir o objetivo:** explicar qual pergunta precisa ser respondida;
2. **confirmar autorização:** verificar escopo, ambiente, artefatos e limites;
3. **preservar evidências:** guardar cópias, versões e hashes dos arquivos;
4. **coletar informações:** ler documentação, observar logs e identificar componentes;
5. **formular hipóteses:** escrever o que parece acontecer e por quê;
6. **fazer análise estática:** examinar código, binários, configurações e dependências;
7. **fazer análise dinâmica:** executar testes controlados e observar o comportamento;
8. **comparar resultados:** verificar quais hipóteses foram confirmadas ou refutadas;
9. **documentar o modelo:** registrar fluxos, formatos, limitações e grau de confiança;
10. **validar com testes:** repetir experimentos para garantir que a conclusão não foi acidental.

Separar observação de interpretação é importante. “O sistema retorna 404 quando o campo falta” é uma observação. “O sistema considera o campo obrigatório” é uma interpretação que precisa ser confirmada em outros cenários.

## Exemplo: entender uma API autorizada

Imagine que uma equipe precisa integrar um [[Frontend]] com um serviço antigo. A documentação está incompleta, mas existe autorização para usar um ambiente de testes.

Um fluxo seguro seria:

1. listar as rotas conhecidas e os usuários de teste;
2. enviar requisições pequenas e não destrutivas;
3. observar códigos de status, cabeçalhos e formatos de resposta;
4. registrar quais campos são obrigatórios;
5. comparar respostas para entradas válidas e inválidas;
6. documentar o contrato descoberto;
7. criar [[Testes]] automatizados para proteger a integração.

Uma chamada de leitura poderia ser:

```bash
curl -i https://api-teste.exemplo/animais/42
```

O [[cURL]] pode mostrar a resposta HTTP e ajudar a confirmar o formato observado. O endereço é apenas ilustrativo: nunca faça chamadas contra sistemas de terceiros sem autorização.

O resultado pode ser uma tabela como esta:

| Observação | Hipótese | Como validar |
| --- | --- | --- |
| `GET /animais/42` retorna `200` | o animal existe e pode ser consultado | testar um identificador inexistente |
| campo `id` aparece sempre | o identificador faz parte da resposta | comparar vários registros |
| `POST` sem `nome` retorna `400` | `nome` é obrigatório | repetir com nome vazio e válido |
| resposta contém `talhaoId` | existe relação com um talhão | consultar o recurso relacionado |

Esse trabalho ajuda a criar um cliente confiável sem inventar regras a partir de uma única resposta.

## Exemplo: analisar uma aplicação Java

Quando existe um arquivo `.jar` ou uma aplicação [[Quarkus]], a análise pode começar pela identificação das dependências, classes públicas, configurações e pontos de entrada.

Perguntas úteis:

- quais endpoints são expostos;
- quais classes representam recursos e serviços;
- quais configurações vêm de variáveis de ambiente;
- qual banco de dados é acessado;
- como são tratados erros e autenticação;
- quais bibliotecas aparecem no pacote;
- quais chamadas acontecem quando uma rota é executada.

O objetivo não é presumir que o nome de uma classe conta toda a história. Uma classe chamada `Service` pode ter responsabilidades demais, enquanto uma classe com nome genérico pode conter uma regra importante.

Se o sistema antigo usa um [[Design Pattern]], reconhecer a ideia pode ajudar a organizar a investigação. Ainda assim, não se deve forçar todo código a encaixar em um padrão conhecido.

## Engenharia reversa de protocolos

Um protocolo é um conjunto de regras para trocar mensagens. Para estudá-lo, observe:

- como uma sessão começa e termina;
- quais mensagens são enviadas;
- ordem dos campos;
- tipos e tamanhos dos valores;
- códigos de sucesso e erro;
- limites de tempo e tentativas;
- identificação e autenticação;
- comportamento quando uma mensagem é repetida ou perdida.

Um diagrama de sequência pode ajudar:

```text
Dispositivo       Serviço             Banco
     |                |                  |
     |--- leitura --->|                  |
     |                |--- valida ------>|
     |                |<-- confirmação --|
     |<-- resposta ---|                  |
```

Ao estudar uma comunicação, não altere dados reais nem tente contornar controles. Use dispositivos de teste, servidores locais ou uma cópia isolada quando possível.

## Engenharia reversa e segurança

Na área de segurança, a engenharia reversa ajuda a investigar vulnerabilidades, analisar malware e entender o impacto de um incidente.

Boas práticas:

- obter autorização documentada e definir o escopo;
- usar uma máquina virtual ou rede isolada;
- trabalhar com cópias dos arquivos originais;
- calcular um hash, como [[SHA-256]], para identificar alterações;
- não executar malware em um computador de uso pessoal;
- bloquear acesso desnecessário à internet;
- não enviar amostras confidenciais para serviços públicos;
- registrar evidências, horários e ferramentas usadas;
- proteger relatórios e informações encontradas;
- comunicar vulnerabilidades de forma responsável.

Uma análise autorizada pode revelar que um programa armazena segredos, confia demais em dados do usuário ou usa uma biblioteca vulnerável. A descoberta deve ser tratada como parte de um processo de [[Segurança]], não como autorização para explorar pessoas ou sistemas.

## Limites legais e éticos

Antes de analisar software, hardware ou tráfego de rede de outra pessoa, confirme:

- quem é o proprietário;
- qual é a autorização;
- quais ambientes e horários estão no escopo;
- quais técnicas são permitidas;
- como os dados serão armazenados e apagados;
- quem receberá o relatório;
- quais condições existem para divulgar uma falha.

Licenças, contratos, leis e regras de cada país podem tratar engenharia reversa de maneiras diferentes. Em caso de dúvida, procure orientação jurídica. Uma finalidade de estudo não transforma automaticamente uma atividade sem autorização em atividade permitida.

Nunca use engenharia reversa como justificativa para:

- acessar uma conta ou sistema sem permissão;
- roubar credenciais, dados pessoais ou propriedade intelectual;
- contornar autenticação, controle de acesso ou pagamento;
- derrubar um serviço ou alterar dados de produção;
- distribuir malware ou instruções para causar dano.

## Boas práticas

- definir uma pergunta concreta antes de começar;
- registrar todas as hipóteses e evidências;
- trabalhar sempre em cópias quando isso for possível;
- usar [[Docker]] ou [[Docker Compose]] para reproduzir um ambiente controlado;
- versionar scripts e documentação com [[Git]];
- calcular hashes para acompanhar a integridade dos artefatos;
- combinar análise estática e dinâmica;
- validar conclusões com entradas diferentes;
- manter dados reais separados do laboratório;
- proteger tokens, chaves e informações pessoais;
- documentar limitações e incertezas;
- transformar descobertas em testes ou documentação útil;
- apagar artefatos sensíveis quando o período de retenção terminar.

## O que a engenharia reversa não garante?

Mesmo com bastante esforço, talvez não seja possível descobrir:

- o nome original de variáveis e classes;
- comentários e decisões que não ficaram no artefato;
- todos os caminhos de execução;
- o motivo histórico de uma regra;
- partes protegidas por hardware ou criptografia;
- comportamentos que dependem de dados ou serviços indisponíveis.

Por isso, uma conclusão deve indicar o nível de confiança. É melhor escrever “observado nos cenários A e B” do que afirmar que uma regra é universal sem evidência suficiente.

## Resumo

> **Reverse engineering é o estudo sistemático de um sistema existente para entender sua estrutura e seu comportamento.**

Ela pode apoiar manutenção, integração, segurança e aprendizado. O trabalho deve ser autorizado, controlado, documentado e orientado a uma finalidade legítima.
