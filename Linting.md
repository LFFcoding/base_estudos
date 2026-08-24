# O que é linting?

**Linting** é a análise automática do código para encontrar problemas antes que ele chegue à execução ou à produção.

Um programa chamado **linter** lê os arquivos e procura sinais de erros, inconsistências, código difícil de entender, regras de estilo não seguidas e alguns riscos conhecidos.

Uma analogia é uma revisão automática de um texto. O linter não escreve o texto por você, mas aponta palavras repetidas, erros de formatação e trechos que podem causar confusão. A pessoa desenvolvedora decide como corrigir o problema.

## O que um linter pode encontrar?

Dependendo da linguagem e da configuração, ele pode detectar:

- variáveis declaradas e nunca utilizadas;
- imports que não são usados;
- código que nunca será alcançado;
- nomes fora do padrão da equipe;
- linhas muito longas;
- espaços e aspas inconsistentes;
- comparações suspeitas;
- possíveis erros conhecidos da linguagem;
- uso de práticas desaconselhadas;
- alguns padrões de vulnerabilidade ou código inseguro.

Por exemplo, este código possui uma variável que não é usada:

```ts
function saudacao(nome: string): string {
  const mensagem = "Olá";
  return `Olá, ${nome}!`;
}
```

Um linter pode avisar que `mensagem` foi criada, mas não participa do resultado. Remover a variável deixa a intenção do código mais clara.

## Linting é análise estática

Linting é uma forma de **análise estática (*static analysis*)**: o código é examinado sem que a aplicação precise ser executada completamente.

Isso permite encontrar problemas cedo, quando corrigir ainda costuma ser barato. O linter pode ser usado no editor, no terminal ou em um pipeline de [[CI-CD]].

## Linter, formatter, compilador e testes

Essas ferramentas se complementam, mas fazem trabalhos diferentes:

| Ferramenta | Principal objetivo |
|---|---|
| **Linter** | Encontrar padrões problemáticos e descumprimento de regras. |
| **Formatter** | Ajustar automaticamente a aparência do código. |
| **Compilador** | Transformar o código ou verificar se ele pode ser compilado. |
| **Verificador de tipos** | Conferir se os valores usados combinam com os tipos declarados. |
| **Teste** | Verificar se o programa apresenta o comportamento esperado. |

Um formatter pode quebrar uma linha comprida, mas não necessariamente percebe que a regra de negócio está errada. Um linter pode apontar um problema, mas não substitui os [[Testes]].

## Exemplo com JavaScript e TypeScript

Em projetos com [[JavaScript]] ou [[TypeScript]], o ESLint é uma ferramenta comum de linting. Um projeto pode ter scripts parecidos com estes no `package.json`:

```json
{
  "scripts": {
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  }
}
```

Os comandos podem ser executados assim:

```bash
npm run lint
npm run typecheck
```

O primeiro executa as regras do linter. O segundo verifica os tipos do TypeScript sem gerar arquivos. A configuração exata depende do projeto e das ferramentas instaladas.

Uma regra pode impedir uma comparação que quase sempre indica erro:

```ts
const idade: number = 18;

if (idade === "18") {
  console.log("Maior de idade");
}
```

O valor `idade` é um número, mas `"18"` é um texto. O [[TypeScript]] pode apontar essa incompatibilidade, e as regras do linter podem ajudar a evitar comparações inadequadas.

## Exemplo com Java e Maven

Em uma aplicação [[Java]] organizada pelo [[Maven]], ferramentas como Checkstyle, PMD ou SpotBugs podem ser configuradas no `pom.xml`.

Quando essas verificações fazem parte do ciclo do Maven, um comando como este pode executar as validações:

```bash
mvn verify
```

O comando não significa automaticamente que todo projeto Java terá linting. Ele executará as verificações configuradas naquele projeto. Em um serviço [[Quarkus]], por exemplo, a equipe pode fazer o linting participar da validação antes de gerar o artefato da aplicação.

## Linting no frontend e no backend

No [[Frontend]], o linting pode encontrar imports não utilizados, propriedades incorretas em componentes e padrões problemáticos em código [[React]] ou [[Next.js]].

No [[Backend]], ele pode verificar organização de classes, tratamento de exceções, nomes, dependências desnecessárias e chamadas que aumentam o risco de erros. Projetos [[NestJS]] em TypeScript também costumam configurar regras para controllers, services e módulos.

O linter não entende automaticamente todas as regras do negócio. Ele pode apontar que uma variável está sem uso, mas não sabe sozinho se um pedido deve ou não ser cancelado.

## Erro, aviso e regra desativada

As ferramentas normalmente classificam problemas como:

- **erro (*error*):** deve ser corrigido para a verificação passar;
- **aviso (*warning*):** merece atenção, mas pode não impedir o processo;
- **informação:** apenas orienta a pessoa desenvolvedora.

Uma equipe deve definir quais problemas bloqueiam o merge. Se tudo for apenas aviso, muitos avisos podem ser ignorados. Se tudo bloquear desde o primeiro dia, adotar linting em um projeto antigo pode ser difícil.

Quando uma exceção for necessária, prefira uma desativação pequena e justificada:

```ts
// Necessário porque a API externa exige este nome exatamente.
// eslint-disable-next-line no-unused-vars
const user_id = resposta.user_id;
```

Não desative o linter inteiro para esconder um problema. A exceção deve ser local, rara e explicada.

## Linting no editor e no terminal

Uma extensão do editor pode sublinhar o problema enquanto o código é escrito. Isso dá feedback rápido, mas não deve ser a única verificação, pois cada pessoa pode ter extensões ou configurações diferentes.

Os comandos do projeto são mais confiáveis para a equipe:

```bash
npm run lint
```

ou, em um projeto Java com verificações configuradas:

```bash
mvn verify
```

O resultado deve ser reproduzível em qualquer máquina preparada para o projeto.

## Linting no Git e no CI/CD

O linting pode ser executado em dois momentos:

1. **Antes do commit:** fornece retorno rápido e evita registrar problemas simples.
2. **No [[CI-CD]]:** confirma que a regra também foi executada em um ambiente automatizado.

Um fluxo comum é:

```text
linting -> verificação de tipos -> [[Testes]] -> build
```

O linting costuma ser colocado no começo porque é rápido. Se falhar, a equipe pode corrigir o problema antes de gastar tempo com etapas mais demoradas.

## Boas práticas

1. **Use uma configuração versionada.** O arquivo de regras deve fazer parte do projeto para que todos usem o mesmo padrão.
2. **Comece com regras úteis.** Muitas regras difíceis podem gerar rejeição ou excesso de falsos positivos.
3. **Corrija os problemas novos.** Em um projeto antigo, pode ser melhor registrar o estado atual e impedir que novos problemas apareçam.
4. **Automatize o comando.** Deixe o linting disponível em um script simples, como `npm run lint`.
5. **Execute no CI/CD.** A validação automatizada evita depender apenas do editor local.
6. **Não confunda estilo com correção.** Uma regra de espaços é diferente de uma regra que pode prevenir um bug.
7. **Revise regras desativadas.** Exceções antigas podem deixar de ser necessárias.
8. **Use formatter quando fizer sentido.** Formatação automática reduz discussões sobre detalhes mecânicos; o linter pode se concentrar em problemas de qualidade.
9. **Não ignore pastas geradas.** Dependências instaladas e arquivos compilados normalmente não devem ser analisados como código da equipe.
10. **Não confie somente no linter para segurança.** Faça revisão, testes e análises específicas quando o risco exigir.

## O que linting não garante?

Linting não garante que:

- a regra de negócio está correta;
- o banco de dados está bem modelado;
- a aplicação está rápida;
- a autorização está correta;
- uma integração externa está funcionando;
- não existe nenhuma vulnerabilidade;
- a resposta de uma [[Requisição]] está no formato esperado.

Ele é uma camada de proteção, não uma prova de que o sistema inteiro está correto. Combine linting com revisão de código, [[Testes]], monitoramento e boas práticas de segurança.

## Resumo

Linting é a verificação automática do código em busca de problemas de qualidade, estilo e possíveis erros. Ele ajuda a encontrar problemas cedo e a manter um padrão entre as pessoas da equipe. Para funcionar bem, deve usar regras versionadas, ser executado no terminal e no [[CI-CD]], e ser combinado com formatter, compilação, testes e revisão humana.

### Veja também

- [[JavaScript]]
- [[TypeScript]]
- [[Java]]
- [[Maven]]
- [[Quarkus]]
- [[NestJS]]
- [[React]]
- [[Next.js]]
- [[Frontend]]
- [[Backend]]
- [[Git]]
- [[CI-CD]]
- [[Testes]]
- [[Requisição]]
