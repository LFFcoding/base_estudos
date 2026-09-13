# O que é o catálogo GoF?

**GoF** significa **Gang of Four** (Gangue dos Quatro). É o apelido dado aos quatro autores do livro *Design Patterns: Elements of Reusable Object-Oriented Software*, publicado em 1994:

- Erich Gamma;
- Richard Helm;
- Ralph Johnson;
- John Vlissides.

O livro reuniu 23 padrões de projeto (*design patterns*) para problemas recorrentes de programação orientada a objetos. Esses padrões não são bibliotecas prontas nem trechos de código para copiar. São modelos de organização de classes, objetos e responsabilidades.

Uma analogia: o catálogo é como um livro de plantas de construção. Ele mostra formatos que costumam funcionar para certos problemas, mas ainda é preciso escolher a planta adequada ao terreno e adaptar os detalhes ao projeto.

## Os três grupos

Os 23 padrões são organizados em três grupos:

1. **Criação (*creational*):** como objetos são criados;
2. **Estrutura (*structural*):** como objetos e classes são combinados;
3. **Comportamento (*behavioral*):** como objetos colaboram e distribuem responsabilidades.

O catálogo não é uma lista de tarefas obrigatórias. Um sistema pode usar poucos padrões, muitos padrões ou nenhum padrão GoF explícito.

## Padrões de criação

| Padrão | Ideia principal | Exemplo de uso |
|---|---|---|
| **Abstract Factory** | cria famílias de objetos relacionados e compatíveis | tema claro/escuro, componentes por plataforma |
| **Builder** | monta um objeto complexo passo a passo | configuração de requisição, relatório ou pedido |
| **Factory Method** | deixa uma extensão decidir qual produto criar | processadores ou documentos de tipos diferentes |
| **Prototype** | cria objetos copiando um protótipo existente | configurações ou objetos já preparados |
| **Singleton** | controla uma instância compartilhada | recurso único, com cuidado para não criar estado global excessivo |

### Abstract Factory

Cria famílias de objetos que devem funcionar juntas. Por exemplo, uma fábrica de tema claro cria um botão claro e uma janela clara; uma fábrica de tema escuro cria os dois componentes escuros.

O benefício é evitar misturar objetos incompatíveis. O custo é criar várias interfaces e classes. [[Factory Pattern]] explica a família de factories com mais detalhes.

### Builder

Se um objeto tem muitos parâmetros opcionais ou precisa ser montado em etapas, o Builder separa a construção do resultado final:

```text
Builder
 ├── define cliente
 ├── adiciona itens
 ├── define endereço
 └── constrói Pedido
```

Ele pode tornar o código mais legível, mas pode ser exagero para objetos pequenos.

### Factory Method

Define um ponto de criação que pode ser especializado. O fluxo principal usa o produto por meio de uma abstração e uma subclasse decide qual implementação instanciar.

Não confunda Factory Method com qualquer método chamado `create`. O padrão envolve uma decisão de criação e uma estrutura que permite variar o produto.

### Prototype

Cria um novo objeto copiando outro. É útil quando construir o objeto do zero é caro ou quando existem protótipos configurados:

```text
protótipo de inimigo → cópia → ajusta posição e vida
```

É preciso decidir se a cópia é rasa (*shallow copy*) ou profunda (*deep copy*). Copiar apenas referências internas pode fazer objetos aparentemente independentes compartilharem estado sem querer.

### Singleton

Garante ou controla uma instância compartilhada. É fácil de acessar, mas pode esconder dependências, dificultar testes e criar estado global.

Antes de usar Singleton, considere uma instância normal recebida por injeção de dependência. Uma configuração ou um cliente compartilhado pode ter ciclo de vida controlado sem precisar de acesso global.

## Padrões estruturais

| Padrão | Ideia principal | Exemplo de uso |
|---|---|---|
| **Adapter** | converte uma interface em outra que o cliente entende | integrar uma biblioteca externa |
| **Bridge** | separa abstração e implementação para variar as duas | notificações por canais e provedores |
| **Composite** | trata objetos individuais e grupos de forma uniforme | árvore de arquivos ou componentes de interface |
| **Decorator** | acrescenta comportamento envolvendo um objeto | cache, log, validação ou retry |
| **Facade** | oferece uma entrada simples para um subsistema complexo | checkout ou caso de uso de backend |
| **Flyweight** | compartilha dados comuns para economizar memória | muitos objetos com propriedades repetidas |
| **Proxy** | representa outro objeto e controla seu acesso | lazy loading, cache, acesso remoto |

### Adapter

O Adapter traduz um contrato para outro:

```text
Sistema → interface esperada → Adapter → API externa
```

Ele é útil quando uma biblioteca ou serviço externo tem nomes, formatos ou métodos incompatíveis com o código interno. Não é necessário mudar toda a aplicação para seguir o formato do fornecedor.

### Bridge

O Bridge separa duas dimensões que poderiam gerar muitas subclasses. Por exemplo, uma notificação pode variar por tipo de mensagem e por provedor de entrega. Separar essas dimensões permite combinar as opções sem criar uma classe para cada par.

### Composite

O Composite organiza estruturas em árvore. Um arquivo e uma pasta podem oferecer a mesma operação `tamanho()`, mesmo que a pasta calcule o total dos seus filhos.

Ele simplifica o código consumidor, mas é importante definir quais operações fazem sentido para folhas e grupos.

### Decorator

O Decorator envolve um objeto que possui o mesmo contrato e acrescenta uma responsabilidade:

```text
serviço original
    ↓ Decorator de log
        ↓ Decorator de cache
            ↓ Decorator de métricas
```

O objeto pode ser decorado sem alterar sua classe original. Muitos decorators encadeados podem dificultar a investigação do fluxo, então documente a ordem.

### Facade

[[Facade Pattern]] oferece uma operação simples para coordenar várias classes de um subsistema. Um checkout pode esconder validação, estoque, pagamento, pedido e notificação atrás de `finalizarCompra()`.

A fachada deve orquestrar e delegar. Ela não deve virar um objeto que contém todas as regras de todos os domínios.

### Flyweight

O Flyweight compartilha a parte repetida de muitos objetos. Em um jogo com milhares de árvores, por exemplo, o modelo e a textura podem ser compartilhados, enquanto posição e escala permanecem específicas de cada instância.

Separe corretamente:

- **estado intrínseco:** pode ser compartilhado;
- **estado extrínseco:** depende de cada uso e deve ser fornecido pelo contexto.

### Proxy

O Proxy apresenta uma interface semelhante à do objeto real e controla o acesso. Pode atrasar o carregamento, verificar autorização, registrar logs ou encaminhar chamadas remotas.

Um Proxy não é automaticamente um mecanismo de segurança. A autorização precisa ser validada de forma correta e no local adequado.

## Padrões comportamentais

| Padrão | Ideia principal | Exemplo de uso |
|---|---|---|
| **Chain of Responsibility** | passa uma solicitação por uma cadeia de handlers | validações e filtros |
| **Command** | transforma uma ação em um objeto | fila, histórico e desfazer |
| **Interpreter** | interpreta uma pequena linguagem ou gramática | filtros ou expressões simples |
| **Iterator** | percorre uma coleção sem expor sua estrutura | travessia de lista ou árvore |
| **Mediator** | centraliza a comunicação entre objetos | coordenação de componentes |
| **Memento** | salva e restaura o estado sem expor detalhes | desfazer e checkpoints |
| **Observer** | notifica interessados quando algo muda | eventos e atualização de telas |
| **State** | muda o comportamento conforme o estado interno | pedido, personagem ou conexão |
| **Strategy** | troca algoritmos por meio de um contrato | desconto, frete ou ordenação |
| **Template Method** | fixa um fluxo e deixa etapas para subclasses | importação e processamento |
| **Visitor** | adiciona operações a uma estrutura de objetos | árvores e compiladores |

### Chain of Responsibility

Uma solicitação passa por handlers até ser tratada ou chegar ao final:

```text
requisição → autenticação → autorização → validação → execução
```

Cada handler decide se trata, rejeita ou encaminha. A cadeia fica flexível, mas uma cadeia longa pode dificultar saber qual componente tomou a decisão.

### Command

O Command transforma uma ação em um objeto. Isso permite colocar ações em filas, registrar histórico, implementar desfazer e executar depois.

```java
interface Comando {
    void executar();
}
```

Uma ação de “adicionar item” pode ser armazenada e executada por um worker. O comando deve carregar somente os dados necessários e tratar repetição com cuidado.

### Interpreter

O Interpreter representa uma gramática e interpreta expressões. Pode servir para filtros, regras pequenas ou linguagens específicas do domínio.

Para linguagens grandes, construir um interpretador manual pode ser complexo. Avalie parser e ferramentas apropriadas antes de transformar muitos `if`s em uma árvore difícil de manter.

### Iterator

O Iterator permite percorrer uma coleção sem expor como ela armazena seus elementos. Um `for-each` normalmente usa uma forma desse padrão por baixo.

Ele separa a lógica de travessia da estrutura interna, mas a coleção deve definir claramente se a alteração durante a iteração é permitida.

### Mediator

O Mediator centraliza a comunicação entre objetos que, de outra forma, teriam muitas referências entre si. Um formulário pode avisar um mediator que um campo mudou, e o mediator decide quais componentes atualizar.

O mediator reduz ligações diretas, mas pode virar um objeto central grande. Mantenha suas responsabilidades divididas por contexto.

### Memento

O Memento captura um estado e permite restaurá-lo sem expor sua estrutura interna. É comum em desfazer, checkpoints e jogos salvos.

O estado salvo pode ocupar muita memória ou conter dados sensíveis. Defina o que realmente precisa ser guardado e proteja os arquivos de save quando necessário.

### Observer

O Observer permite que um objeto avise vários interessados quando algo acontece. [[Interface abstrata]] e signals de engines de jogos usam ideias parecidas.

Exemplos:

- uma alteração de preço atualiza uma tela;
- um evento de pedido informa um sistema de notificação;
- um botão avisa o componente que deve reagir.

Evite deixar observadores esquecidos ou criar ciclos de dependência. Defina claramente o ciclo de vida das inscrições.

### State

O State encapsula comportamentos associados a estados diferentes:

```text
Pedido
├── Criado: pode ser pago
├── Pago: pode ser enviado
└── Cancelado: não aceita novas alterações
```

Ele evita um único método cheio de condições, mas pode criar muitas classes para estados simples. Use-o quando os estados realmente mudarem regras e transições.

### Strategy

[[Strategy Pattern]] encapsula algoritmos intercambiáveis. Um serviço pode usar estratégias de frete normal, expresso ou grátis sem conhecer as fórmulas.

Strategy geralmente usa composição e injeção, enquanto Template Method usa herança. A escolha depende de o comportamento precisar ser trocado por objeto ou definido por subclasses.

### Template Method

Uma classe-base define o esqueleto do algoritmo e chama métodos que as subclasses implementam:

```text
processar()
 ├── validar()       ← comum
 ├── lerDados()      ← varia
 └── salvarResultado()← varia
```

Ele combina bem com [[Classe abstrata]], mas herança excessiva pode deixar a evolução difícil. Proteja o fluxo comum e deixe pontos de extensão claros.

### Visitor

O Visitor separa uma operação da estrutura de objetos sobre a qual ela atua. É útil quando a estrutura é estável e novas operações aparecem com frequência.

O custo é que adicionar um novo tipo de elemento pode exigir alterar todos os visitors. Use-o quando esse equilíbrio fizer sentido.

## Como os padrões se combinam

Padrões raramente aparecem isolados:

```text
Factory → cria uma Strategy
Facade  → coordena vários serviços
Decorator → adiciona log e cache
Proxy   → controla acesso ao serviço
Observer → publica mudanças
```

Uma composição possível em um [[Backend]] seria:

1. uma Facade recebe um caso de uso;
2. uma Factory escolhe uma Strategy;
3. um Adapter conversa com um provedor externo;
4. um Decorator registra métricas;
5. um Observer publica um evento;
6. uma fila processa um Command posteriormente.

Combinar padrões pode organizar o sistema, mas também aumenta a quantidade de abstrações. Cada combinação precisa resolver um problema real.

## Como escolher um padrão

Comece pelo problema, não pelo nome do padrão:

1. descreva a mudança ou dificuldade concreta;
2. identifique quem conhece detalhes demais;
3. veja se há uma responsabilidade que pode ser isolada;
4. escolha a menor estrutura que resolve o problema;
5. avalie teste, leitura, desempenho e manutenção;
6. documente por que o padrão foi usado.

Algumas perguntas rápidas:

```text
Precisa criar famílias compatíveis?       → Abstract Factory
Precisa montar algo complexo em etapas?   → Builder
Precisa adaptar uma API incompatível?     → Adapter
Precisa simplificar vários serviços?      → Facade
Precisa acrescentar comportamento?        → Decorator
Precisa trocar um algoritmo?              → Strategy
Precisa mudar comportamento por estado?   → State
Precisa avisar vários interessados?       → Observer
Precisa representar uma ação executável?  → Command
```

Essas perguntas são pistas, não respostas automáticas.

## GoF e princípios SOLID

Os padrões podem ajudar princípios de [[SOLID]], mas não os garantem:

- Strategy pode separar responsabilidades e depender de abstrações;
- Facade pode reduzir acoplamento entre camadas;
- Decorator pode acrescentar comportamento sem alterar a classe original;
- Adapter pode isolar mudanças de um fornecedor externo;
- Template Method pode reutilizar um fluxo comum.

Um padrão aplicado de forma exagerada pode produzir classes demais, dependências escondidas e violações dos mesmos princípios que deveria ajudar.

## GoF e testes

Um padrão bem escolhido deve facilitar os [[Testes]], não apenas deixar o diagrama mais bonito:

- teste cada estratégia ou implementação concreta;
- teste o fluxo da facade;
- teste handlers isoladamente e em cadeia;
- teste comandos repetidos e desfeitos;
- teste inscrições e remoções de observers;
- teste estados válidos e transições inválidas;
- use fakes e contratos pequenos para substituir integrações externas.

Se um padrão torna um teste simples impossível, investigue se a estrutura está excessivamente acoplada.

## Padrões e segurança

Nenhum padrão GoF substitui segurança. Um Proxy pode verificar acesso, mas precisa de regras corretas. Uma Facade pode expor um caso de uso, mas deve validar a [[Requisição]]. Um Singleton pode guardar configuração, mas não deve conter segredos sem proteção.

Boas práticas continuam necessárias:

- autenticar e autorizar operações;
- validar dados recebidos;
- não confiar no cliente;
- proteger credenciais e tokens;
- controlar repetição de comandos;
- registrar eventos sem expor informações sensíveis;
- seguir as regras de [[Segurança]].

## Catálogo não é checklist

Estudar os 23 padrões é útil para reconhecer nomes e estruturas, mas não é necessário aplicar todos. Um sistema simples com funções claras pode ser melhor que um sistema cheio de factories, visitors e mediators.

Sinais de excesso:

- uma mudança pequena exige navegar por muitas classes;
- o padrão é explicado com mais esforço que o problema;
- existem interfaces sem implementações alternativas reais;
- a equipe usa nomes de padrões sem entender suas responsabilidades;
- abstrações foram criadas apenas para parecer flexível.

Prefira clareza, coesão e capacidade de mudança. O padrão é uma ferramenta, não um objetivo.

## Resumo dos 23 padrões

### Criação

Abstract Factory, Builder, Factory Method, Prototype e Singleton.

### Estrutura

Adapter, Bridge, Composite, Decorator, Facade, Flyweight e Proxy.

### Comportamento

Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method e Visitor.

## Em uma frase

**O catálogo GoF reúne 23 modelos clássicos para organizar a criação, a estrutura e o comportamento de objetos, ajudando a resolver problemas recorrentes sem substituir o raciocínio sobre o contexto.**

### Veja também

- [[Design Pattern]]
- [[Factory Pattern]]
- [[Facade Pattern]]
- [[Strategy Pattern]]
- [[Classe abstrata]]
- [[Interface abstrata]]
- [[Polimorfismo]]
- [[SOLID]]
- [[Backend]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Testes]]
- [[Segurança]]
