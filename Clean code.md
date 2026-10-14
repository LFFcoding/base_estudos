# O que é clean code?

**Clean code** significa **código limpo**: código escrito de um jeito que outras pessoas consigam entender, verificar e alterar com segurança.

Uma analogia: código é como uma receita. Uma receita pode até produzir um bom prato, mas, se os passos forem confusos e os ingredientes tiverem nomes vagos, outra pessoa terá dificuldade para repeti-la. Código limpo deixa as intenções e as regras mais fáceis de acompanhar.

“Limpo” não quer dizer que o código seja curto, sofisticado ou cheio de padrões. Quer dizer que ele comunica sua intenção, tem complexidade adequada e pode evoluir sem surpresas desnecessárias.

## Por que legibilidade importa?

Software costuma ser lido e alterado muitas vezes depois de ser escrito. Código claro ajuda a:

- encontrar onde uma regra está implementada;
- corrigir defeitos com menos risco;
- integrar pessoas novas à equipe;
- revisar mudanças;
- testar comportamentos;
- reduzir interpretações diferentes sobre a mesma regra.

O código precisa ser claro para quem o mantém, inclusive a própria pessoa autora alguns meses depois. Isso vale para o [[Backend]], o [[Frontend]], scripts, consultas e configurações.

## Exemplo: nomes que explicam a intenção

Nomes genéricos escondem o significado:

```java
int x = 4;
double y = x * 0.9;
```

Nomes melhores dão contexto:

```java
int quantidadeDeItens = 4;
double totalComDesconto = quantidadeDeItens * 0.9;
```

O segundo exemplo ainda pode precisar de uma regra mais adequada para dinheiro, mas deixa mais claro o papel de cada valor. Em sistemas financeiros, use tipos e operações apropriados para valores monetários, em vez de depender de `double` sem avaliar arredondamentos.

Escolha nomes que revelem o propósito, não o tipo ou a implementação quando isso não ajudar. Evite abreviações que só a pessoa autora entende.

## Funções com uma intenção clara

Uma função fica mais fácil de entender quando seu nome e seus parâmetros mostram o que ela faz:

```java
boolean podeCancelar(Pedido pedido) {
    return pedido.status() == Status.CRIADO;
}
```

Compare com uma condição repetida e espalhada:

```java
if (pedido.status() == Status.CRIADO) {
    // cancelar
}
```

Extrair uma regra para uma função nomeada ajuda quando a lógica tem significado próprio ou é usada em mais de um lugar. Não é necessário criar uma função para cada linha: abstrações pequenas demais também podem esconder o fluxo.

Boas funções tendem a:

- fazer uma tarefa coerente;
- ter poucos parâmetros fáceis de interpretar;
- evitar efeitos colaterais inesperados;
- tratar erros de forma explícita;
- ser simples de testar.

## Reduzir complexidade sem esconder o fluxo

Condições muito aninhadas podem dificultar a leitura:

```java
if (usuario != null) {
    if (usuario.ativo()) {
        if (usuario.temPermissao()) {
            executarAcao();
        }
    }
}
```

Uma forma de tornar a regra mais direta é usar retornos antecipados:

```java
if (usuario == null || !usuario.ativo() || !usuario.temPermissao()) {
    return;
}

executarAcao();
```

Essa forma só é adequada se o comportamento continuar claro e correto. O objetivo é mostrar o caminho principal sem criar truques ou esconder condições importantes.

## Evitar duplicação com discernimento

Duplicação pode causar um problema quando a mesma regra precisa ser corrigida em vários lugares. Se uma regra de cálculo de frete aparecer em três telas, por exemplo, as telas podem começar a produzir valores diferentes.

Mas nem toda repetição deve virar uma abstração compartilhada. Dois trechos parecidos podem representar regras diferentes que mudarão por motivos distintos. Antes de extrair código, confirme se existe uma regra realmente comum.

Uma boa extração dá um nome útil à ideia. Uma função genérica chamada `processar()` pode apenas mover a confusão para outro lugar.

## Comentários e documentação

Prefira código cujo nome e estrutura expliquem o que está acontecendo. Comentários são úteis para explicar contexto, restrições ou motivos que não podem ser expressos claramente no código.

Comentário pouco útil:

```java
// Soma um ao contador
contador++;
```

Comentário mais útil:

```java
// A API externa limita cada lote a 100 registros.
int tamanhoDoLote = 100;
```

Comentários podem ficar desatualizados. Quando a implementação mudar, revise também a explicação. Documente decisões importantes em um [[ADR]] quando o motivo afetar a arquitetura.

## Tratar erros com clareza

Código limpo não esconde falhas nem transforma todos os erros em uma resposta genérica. Ele comunica o que deu errado e preserva informação suficiente para a equipe diagnosticar o problema.

- valide dados externos antes de usá-los;
- use erros apropriados para cada situação;
- não capture uma exceção apenas para ignorá-la;
- não mostre detalhes internos ou segredos a quem usa a aplicação;
- registre contexto útil sem incluir senhas, tokens ou dados pessoais desnecessários.

Esses cuidados ajudam a manter uma [[Segurança]] adequada e tornam os logs mais úteis.

## Testes e refatoração

Código limpo é mais fácil de verificar, e testes ajudam a mudar código com confiança. Os [[Testes]] devem descrever comportamentos importantes, não depender de detalhes internos que podem mudar sem afetar o resultado.

**Refatoração (*refactoring*)** é reorganizar o código sem alterar seu comportamento observável. Um fluxo seguro costuma ser:

1. entender e registrar o comportamento atual;
2. ter testes para as regras importantes;
3. fazer uma mudança pequena;
4. executar os testes e as verificações;
5. revisar o resultado com `git diff`;
6. registrar a alteração no [[Git]].

Quando não há testes, uma mudança pode parecer apenas organizacional e ainda alterar uma regra de negócio por acidente.

## Princípios e ferramentas relacionados

O [[SOLID]] oferece princípios para organizar responsabilidades e dependências. [[Design Pattern|Design patterns]] descrevem soluções reutilizáveis para problemas recorrentes. Ambos podem ajudar, desde que resolvam uma necessidade real.

O [[Linting]] automatiza verificações de estilo e alguns padrões problemáticos. Ele ajuda a manter consistência, mas não decide se os nomes comunicam bem a regra de negócio nem se a arquitetura está simples.

Em [[Java]], [[JavaScript]] e [[TypeScript]], os detalhes de sintaxe mudam, mas nomes claros, funções focadas, tratamento explícito de erros e testes úteis continuam valendo.

## Boas práticas

- prefira nomes que expressem a intenção do domínio;
- organize o código para que o caminho principal seja fácil de seguir;
- mantenha funções e classes com responsabilidades coerentes;
- elimine duplicação de regras, sem criar abstrações prematuras;
- use comentários para registrar motivos e restrições, não para repetir o código;
- trate erros de forma visível e apropriada;
- mantenha testes próximos das regras importantes;
- faça refatorações pequenas e revise o diff;
- use [[Linting]] e formatação automática para decisões mecânicas;
- siga os padrões do projeto para que arquivos semelhantes sejam previsíveis;
- escolha simplicidade em vez de abstrações que não resolvem um problema presente.

## Resumo

> **Clean code é código que deixa suas intenções claras e pode ser alterado com segurança.**

Ele não depende de uma fórmula única. A equipe melhora a clareza com bons nomes, responsabilidades coerentes, testes úteis, revisão e decisões proporcionais à complexidade do problema.
