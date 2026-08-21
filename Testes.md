# Testes

**Testes de software** são verificações feitas para descobrir se uma aplicação se comporta como esperado. Eles podem ser executados manualmente ou por código e ajudam a encontrar problemas antes que cheguem às pessoas usuárias.

Um teste não prova que um sistema não possui nenhum defeito. Ele verifica comportamentos escolhidos pela equipe e aumenta a confiança de que esses comportamentos continuam funcionando.

## Uma analogia

Antes de entregar uma bicicleta, alguém pode apertar os parafusos, testar os freios, girar as rodas e fazer um pequeno percurso.

Cada verificação encontra um tipo de problema. Uma aplicação também precisa de verificações em diferentes níveis: uma regra pequena, a comunicação com o [[Banco de dados|banco]], uma [[Requisição]] HTTP e o fluxo completo usado pela pessoa.

## O que um teste verifica?

Um teste normalmente define:

- um cenário inicial;
- uma ação;
- um resultado esperado;
- uma forma de comparar o resultado real com o esperado.

Um formato comum é **AAA**:

1. **Arrange**: preparar os dados e o ambiente;
2. **Act**: executar a ação que será testada;
3. **Assert**: conferir o resultado.

Exemplo em pseudocódigo:

```text
Arrange: criar um carrinho vazio
Act: adicionar um produto de preço 10
Assert: o total do carrinho deve ser 10
```

Separar essas partes deixa o teste mais fácil de ler e de corrigir quando falhar.

## Principais tipos de teste

### Teste unitário (*unit test*)

Verifica uma unidade pequena e isolada, como uma função, método ou regra de negócio. Deve ser rápido e não depender de rede, banco ou horário real.

Exemplos:

- calcular o total de um pedido;
- validar uma senha;
- transformar um objeto em [[JSON]];
- verificar se uma data está em um período permitido.

### Teste de integração (*integration test*)

Verifica se duas ou mais partes funcionam juntas. Pode testar o [[Backend]] com o [[PostgreSQL]], uma camada de persistência com o banco ou um recurso HTTP com seus componentes internos.

Esse teste é mais próximo da realidade que um teste unitário, mas costuma ser mais lento e exigir preparação de ambiente.

### Teste de componente ou interface

Verifica uma parte visual do [[Frontend]], como um formulário, menu ou contador. Pode conferir se a interface mostra o texto correto, reage a um clique e exibe uma mensagem de erro.

No [[React]] e no [[Next.js]], é comum simular ações de uma pessoa e observar o resultado visível, em vez de testar detalhes internos do componente.

### Teste de ponta a ponta (*end-to-end* ou E2E)

Simula um fluxo completo, geralmente pelo navegador:

1. abrir a página de login;
2. preencher usuário e senha;
3. enviar o formulário;
4. verificar a tela inicial;
5. criar um registro;
6. conferir o resultado.

Esse tipo de teste dá uma visão próxima da experiência real, mas é mais lento, mais caro de manter e pode falhar por problemas no navegador, rede ou ambiente.

### Teste de contrato (*contract test*)

Verifica se dois sistemas concordam sobre o formato da comunicação. Por exemplo, confirma se o [[Backend]] continua enviando os campos que o [[Frontend]] espera em uma resposta [[JSON]].

Ele ajuda quando existem vários serviços ou equipes que dependem da mesma interface.

### Teste de fumaça (*smoke test*)

É uma verificação rápida das funções essenciais. Depois de publicar uma versão, pode conferir se a aplicação inicia, responde a uma rota básica e consegue acessar dependências importantes.

Se o smoke test falhar, não faz sentido gastar tempo em testes mais detalhados naquele ambiente.

### Teste de desempenho e carga

Mede como o sistema se comporta com diferentes quantidades de usuários, dados ou requisições.

- **load test**: verifica uma carga esperada;
- **stress test**: aumenta a carga até encontrar um limite;
- **soak test**: mantém a carga por bastante tempo para encontrar problemas graduais.

O objetivo não é apenas descobrir se uma requisição funciona, mas saber quanto tempo demora, quantos erros acontecem e como o sistema reage quando os recursos ficam disputados.

### Teste de segurança

Procura falhas que permitam acesso indevido, alteração de dados ou exposição de informações. Deve verificar autenticação, autorização, validação de entradas, sessões e proteção contra ataques conhecidos.

Testes automatizados ajudam, mas não substituem as práticas descritas em [[Segurança]] nem uma análise especializada quando o risco for alto.

## A pirâmide de testes

Uma estratégia comum é ter muitos testes unitários, uma quantidade menor de testes de integração e poucos testes E2E:

| Camada | Quantidade comum | Velocidade | O que verifica |
| --- | --- | --- | --- |
| Unitário | muitos | alta | regras pequenas e isoladas |
| Integração | quantidade média | média | comunicação entre partes |
| E2E | poucos | baixa | fluxos completos |

A ideia não é seguir números rígidos. A equipe deve escolher os testes de acordo com o risco. Uma regra financeira importante merece mais proteção do que uma função simples de formatação.

## Exemplo de teste unitário em Java

Na stack deste projeto, o [[Java]] pode usar JUnit para testes unitários:

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

class CalculadoraTest {

    @Test
    void deveSomarDoisNumeros() {
        var calculadora = new Calculadora();

        var resultado = calculadora.somar(2, 3);

        assertEquals(5, resultado);
    }
}
```

O teste cria a calculadora, executa `somar(2, 3)` e verifica se o resultado é `5`. Se alguém alterar a regra e quebrar esse comportamento, o teste falhará.

## Exemplo de teste de integração no Quarkus

O [[Quarkus]] pode executar testes que sobem a aplicação e fazem uma requisição para um endpoint. Um exemplo conceitual é:

```java
@QuarkusTest
class SaudacaoResourceTest {

    @Test
    void deveRetornarUmaSaudacao() {
        given()
            .when().get("/saudacoes")
            .then()
            .statusCode(200)
            .body(containsString("Olá"));
    }
}
```

Esse teste verifica mais partes do que um teste unitário: a rota, o recurso HTTP e a resposta. A configuração real depende das bibliotecas de teste do projeto.

O comando `mvn test` executa os testes configurados no projeto [[Maven]]. É importante que os testes possam rodar de forma repetível em uma máquina local e no pipeline.

## Mocks, stubs, fakes e spies

Às vezes, um teste unitário precisa substituir uma dependência externa. Essas substituições são chamadas de **test doubles**:

- **stub**: devolve respostas preparadas;
- **mock**: permite verificar se uma chamada esperada aconteceu;
- **fake**: implementação simplificada, como um repositório em memória;
- **spy**: registra chamadas reais para permitir observação.

Por exemplo, uma regra de negócio pode receber um repositório falso em vez de acessar o PostgreSQL. Isso deixa o teste rápido. Porém, alguns testes de integração ainda devem verificar se o código funciona com o banco real ou com um ambiente equivalente.

Não transforme toda dependência em mock. Testar somente se métodos foram chamados pode fazer o teste passar mesmo quando o comportamento real está errado.

## Testes e banco de dados

Testes que usam banco precisam cuidar dos dados criados:

- use um banco separado do desenvolvimento e da produção;
- prepare o esquema de forma automatizada;
- isole ou limpe os dados entre os testes;
- não dependa da ordem em que os testes são executados;
- use containers temporários ou um ambiente controlado quando fizer sentido;
- teste transações, restrições, consultas e casos de dados vazios.

O [[Docker]] e o [[Docker Compose]] podem ajudar a reproduzir dependências locais, mas iniciar um [[Container]] não garante que os dados do teste estejam isolados ou que o serviço esteja pronto. O teste deve verificar a disponibilidade e preparar o ambiente corretamente.

## O que torna um teste bom?

Um bom teste costuma ser:

- **determinístico**: produz o mesmo resultado com as mesmas condições;
- **isolado**: não depende de outro teste para funcionar;
- **repetível**: pode ser executado várias vezes;
- **rápido**: fornece feedback sem espera desnecessária;
- **legível**: deixa claro o cenário e o comportamento esperado;
- **focado**: verifica uma regra ou comportamento relacionado;
- **representativo**: protege algo importante para quem usa o sistema.

Evite depender de horário real, números aleatórios sem controle, internet pública, banco compartilhado e dados deixados por outros testes. Essas dependências costumam criar **flaky tests**, testes instáveis que às vezes passam e às vezes falham sem alteração no código.

## Cobertura de código

**Cobertura** (*code coverage*) indica quais linhas, métodos ou caminhos foram executados pelos testes. Ela é útil para encontrar áreas sem nenhuma verificação, mas uma porcentagem alta não garante testes bons.

Um teste pode executar uma linha sem verificar se o resultado está correto. Por isso, priorize comportamentos importantes, casos de erro, limites, permissões e fluxos usados pelas pessoas.

## Testes no CI/CD

No fluxo de [[CI-CD]], uma alteração pode passar por etapas como:

1. compilar o código;
2. executar testes unitários;
3. executar testes de integração;
4. verificar estilo, segurança e dependências;
5. construir o artefato;
6. executar smoke tests no ambiente de teste.

O [[GitHub]] pode executar essas etapas quando alguém abre um pull request ou envia um commit. Se um teste essencial falhar, o merge deve ser bloqueado até que o problema seja entendido e corrigido.

## TDD

**TDD** (*Test-Driven Development* ou desenvolvimento orientado a testes) é uma forma de trabalhar em ciclos curtos:

1. escrever um teste que falha;
2. escrever o código mínimo para fazê-lo passar;
3. melhorar a estrutura sem alterar o comportamento;
4. repetir o ciclo.

TDD pode ajudar a pensar no comportamento antes da implementação, mas não é obrigatório para todo código. O mais importante é manter verificações úteis e executáveis.

## Boas práticas

- teste comportamentos e regras importantes, não apenas linhas de código;
- inclua casos normais, casos de erro e valores de limite;
- teste autenticação e autorização em operações protegidas;
- mantenha testes próximos do código que ajudam a proteger;
- use nomes que expliquem o cenário e o resultado esperado;
- corrija testes instáveis em vez de ignorar falhas;
- execute testes localmente antes de abrir um pull request;
- faça o pipeline executar os testes automaticamente;
- revise testes como parte da revisão de código;
- mantenha o ambiente de teste seguro e separado da produção;
- não use dados reais ou segredos nos testes;
- trate cobertura como indicador, não como objetivo isolado.

**Testes são verificações automatizadas ou manuais que aumentam a confiança de que o sistema continua funcionando como esperado.** Eles complementam revisão de código, monitoramento, segurança e a observação do sistema em produção.
