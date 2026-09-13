# O que é Facade Pattern?

**Facade Pattern** (padrão de fachada) oferece uma interface simples para usar um conjunto de classes ou serviços que, internamente, é mais complexo.

Uma analogia: o concierge de um hotel é uma fachada. O hóspede faz um pedido ao concierge, sem precisar telefonar separadamente para o restaurante, o transporte e a lavanderia. O concierge coordena as partes necessárias e devolve uma resposta mais simples.

No software, a fachada funciona como uma porta de entrada organizada:

```text
cliente → Facade → vários componentes do subsistema
```

O cliente conhece a fachada. A fachada conhece os detalhes necessários para coordenar o subsistema.

Facade é um padrão estrutural de [[Design Pattern]]. Ele não elimina a complexidade interna; apenas impede que ela seja espalhada por todos os consumidores.

## O problema que ele resolve

Imagine um checkout que precisa:

1. validar o carrinho;
2. verificar o estoque;
3. reservar os produtos;
4. autorizar o pagamento;
5. criar o pedido;
6. enviar uma confirmação.

Sem uma fachada, cada controller, tela ou cliente pode precisar conhecer todos esses serviços e chamá-los na ordem correta. Isso aumenta o acoplamento e faz com que o mesmo fluxo seja repetido.

Com uma fachada, o cliente chama uma operação de alto nível:

```java
ResultadoCheckout resultado = checkoutFacade.finalizar(carrinho, cliente);
```

O código de uso não precisa conhecer cada etapa interna.

## Exemplo em Java

```java
public final class CheckoutFacade {
    private final EstoqueService estoque;
    private final PagamentoService pagamentos;
    private final PedidoService pedidos;
    private final NotificacaoService notificacoes;

    public CheckoutFacade(
            EstoqueService estoque,
            PagamentoService pagamentos,
            PedidoService pedidos,
            NotificacaoService notificacoes) {
        this.estoque = estoque;
        this.pagamentos = pagamentos;
        this.pedidos = pedidos;
        this.notificacoes = notificacoes;
    }

    public ResultadoCheckout finalizar(
            Carrinho carrinho,
            Cliente cliente) {
        carrinho.validar();
        estoque.reservar(carrinho.itens());
        pagamentos.autorizar(cliente, carrinho.total());

        Pedido pedido = pedidos.criar(cliente, carrinho);
        notificacoes.enviarConfirmacao(pedido);

        return ResultadoCheckout.sucesso(pedido.id());
    }
}
```

Uso:

```java
var resultado = checkoutFacade.finalizar(carrinho, cliente);
```

Nesse exemplo, a fachada coordena o fluxo, mas os serviços continuam responsáveis por suas próprias regras. `EstoqueService` cuida do estoque, `PagamentoService` cuida do pagamento e `PedidoService` cuida da criação do pedido.

O código é ilustrativo: uma aplicação real também precisa definir transação, compensação, idempotência, tratamento de falhas e o que acontece quando uma etapa é concluída e a seguinte falha.

## O que uma Facade deve fazer?

Uma fachada geralmente:

- oferece operações de alto nível;
- coordena chamadas em uma ordem definida;
- esconde detalhes de inicialização e comunicação;
- traduz resultados internos para um formato útil ao cliente;
- centraliza uma entrada de um caso de uso;
- reduz a quantidade de dependências que o consumidor precisa conhecer.

Ela não deve automaticamente assumir todas as regras do negócio. Se a fachada começa a decidir preço, estoque, pagamento, envio e relatórios diretamente, pode virar um **God Object**.

## Facade não é uma classe “faz tudo”

Existe uma diferença importante:

- uma fachada organiza e delega a operação para componentes especializados;
- um God Object concentra dados e regras de muitas áreas em uma única classe.

Uma fachada pode ter várias chamadas internas e ainda ser saudável, desde que o fluxo fique claro e as responsabilidades detalhadas permaneçam nos serviços adequados.

Se o método da fachada ficar enorme, considere dividir o caso de uso, extrair etapas ou criar fachadas menores por contexto.

## Facade no backend

Em um [[Backend]], uma fachada pode representar um caso de uso, como:

- finalizar compra;
- registrar usuário;
- gerar relatório;
- importar dados;
- iniciar uma transferência;
- montar um painel que combina informações de vários serviços.

Um endpoint HTTP pode chamar uma fachada, mas o recurso REST não precisa conhecer todos os repositórios e integrações envolvidos. Uma estrutura comum é:

```text
Requisição HTTP → Resource/Controller → Facade → serviços → banco e integrações
```

O endpoint pode validar o formato básico e transformar a resposta em [[JSON]], enquanto a fachada coordena o caso de uso. Validações de negócio e regras de autorização não devem ser removidas apenas porque existe uma fachada.

## Facade e transações

Quando uma fachada coordena várias alterações, é necessário decidir qual é a fronteira transacional:

- operações locais podem participar de uma mesma transação;
- chamadas externas podem não aceitar rollback;
- uma falha depois de uma chamada externa pode exigir compensação;
- eventos e mensagens podem exigir o [[Outbox Pattern]];
- repetição segura pode exigir uma [[Chave de idempotência]].

Não coloque uma anotação transacional sem entender quais recursos realmente participam da transação. A fachada pode ser um bom lugar para definir o limite do caso de uso, mas isso depende da arquitetura.

## Facade e injeção de dependência

A fachada deve receber suas dependências por construtor:

```java
public final class RelatorioFacade {
    private final DadosService dados;
    private final ExportadorService exportador;

    public RelatorioFacade(
            DadosService dados,
            ExportadorService exportador) {
        this.dados = dados;
        this.exportador = exportador;
    }
}
```

Isso torna as dependências visíveis e facilita substituir serviços por fakes nos [[Testes]]. A prática se relaciona a [[Chamada estática vs injeção CDI]] e pode ser configurada por [[Quarkus]] ou outro [[Framework]].

Uma [[Factory Pattern]] pode criar ou escolher um componente usado pela fachada. A factory decide como construir; a facade oferece uma operação simples e coordena o uso.

## Facade versus outros padrões

### Facade versus Adapter

- **Facade:** simplifica o uso de várias partes de um subsistema;
- **Adapter:** converte uma interface em outra interface compatível.

Uma fachada pode chamar um Adapter, mas os objetivos são diferentes. Adapter resolve incompatibilidade; Facade resolve complexidade de uso.

### Facade versus Proxy

- **Facade:** apresenta operações de alto nível para um subsistema;
- **Proxy:** representa outro objeto e controla o acesso a ele.

Um proxy pode controlar cache, autorização ou acesso remoto. Uma fachada pode orquestrar várias operações e não precisa ter a mesma interface do subsistema.

### Facade versus Mediator

- **Facade:** normalmente é uma entrada externa para um conjunto de componentes;
- **Mediator:** coordena a comunicação entre objetos que poderiam ficar fortemente acoplados entre si.

Uma fachada pode usar um mediator internamente, mas o cliente geralmente enxerga apenas a fachada.

### Facade versus Service Layer

Uma **Service Layer** organiza operações de aplicação. Muitas Service Layers acabam funcionando como fachadas para casos de uso, mas os termos não são perfeitamente equivalentes. Facade destaca a simplificação de um subsistema; Service Layer destaca a organização da lógica de aplicação.

## Facade e Factory juntos

Uma aplicação pode usar os dois padrões:

```text
Facade
 ├── pede à Factory o provedor correto
 ├── coordena o caso de uso
 └── devolve um resultado simples
```

Por exemplo, a fachada de pagamentos pode pedir à factory uma implementação de gateway conforme a configuração. A fachada não precisa conhecer a construção de cada gateway.

## Facade no C# e no C++

O padrão não depende de uma linguagem. Ele pode ser implementado em [[C#]] ou [[C++]] com classes, interfaces, referências e composição.

Em C++, mantenha atenção ao tempo de vida dos serviços e prefira RAII para recursos. Em C#, use interfaces e injeção de dependência quando isso deixar a composição e os [[Testes]] mais simples.

## Facade no frontend

No [[Frontend]], uma fachada pode esconder várias chamadas de API e apresentar uma função de alto nível para um componente:

```text
Tela de painel → DashboardFacade.carregar() → usuários, pedidos e métricas
```

Ela pode combinar respostas, adaptar dados e controlar o fluxo de carregamento. Porém, evite criar uma fachada gigante que contenha o estado de toda a aplicação. Separe por tela, caso de uso ou contexto quando necessário.

## Facade e bibliotecas

Uma [[Biblioteca]] pode oferecer uma fachada para evitar que o consumidor conheça dezenas de classes internas. Uma API pública pequena reduz a quantidade de detalhes que precisam ser documentados e preservados.

Ao criar uma fachada pública, trate sua interface como um contrato: nomes, parâmetros, erros e formato de retorno devem mudar com cuidado.

## Tratamento de erros

Uma fachada deve definir como erros internos aparecem para quem a chama:

- converta exceções técnicas em erros úteis ao caso de uso;
- preserve a causa original para logs e investigação;
- não exponha stack traces ou detalhes de banco ao cliente;
- diferencie erro de validação, indisponibilidade e falha inesperada;
- deixe claro se uma operação foi concluída parcialmente;
- defina se uma repetição é segura.

No caso de mensagens ou chamadas remotas, considere timeout, retry limitado e [[Idempotência]].

## Segurança

Uma fachada não é uma barreira de segurança por si só. Ao expor um caso de uso:

- autentique o chamador;
- autorize cada operação conforme o usuário e o recurso;
- valide entradas vindas de uma [[Requisição]];
- não confie em IDs, valores ou permissões enviados pelo cliente;
- mantenha segredos fora do código e das respostas;
- registre ações importantes sem expor dados sensíveis;
- aplique as regras de [[Segurança]] também nos serviços internos.

Não remova validações dos subsistemas apenas porque existe uma fachada. Uma integração interna também pode ser chamada por outro fluxo ou ser usada incorretamente.

## Testes

Teste a fachada como coordenadora do fluxo:

- confirme que as etapas principais são chamadas na ordem correta;
- verifique que falhas interrompem ou compensam o fluxo esperado;
- teste entradas inválidas e casos-limite;
- use fakes ou mocks para serviços externos;
- escreva testes de integração para confirmar as conexões reais;
- teste idempotência quando a operação puder ser repetida.

O teste da fachada não substitui o teste individual de `EstoqueService`, `PagamentoService` e os demais componentes. Cada camada deve ter testes compatíveis com sua responsabilidade.

## Boas práticas

- dê à fachada um nome ligado ao caso de uso ou ao subsistema;
- mantenha a interface pública pequena;
- receba dependências por construtor;
- delegue regras detalhadas aos componentes especializados;
- defina claramente a fronteira transacional;
- retorne DTOs ou resultados próprios, sem vazar objetos internos;
- trate erros de forma consistente;
- evite métodos gigantes e fachadas com responsabilidades de muitos domínios;
- não transforme uma fachada em um Service Locator;
- documente ordem, efeitos colaterais e possibilidade de repetição;
- teste o fluxo e as falhas importantes;
- mantenha a fachada independente de detalhes desnecessários do subsistema.

## Quando usar

Facade costuma ser uma boa escolha quando:

- um subsistema possui muitas classes e uma sequência difícil de usar;
- vários clientes repetem o mesmo fluxo;
- você quer reduzir dependências entre camadas;
- o formato interno precisa mudar sem afetar todos os consumidores;
- existe um caso de uso claro que pode ser exposto como uma operação.

## Quando evitar

Talvez uma fachada seja desnecessária quando:

- o subsistema já é pequeno e simples;
- ela apenas repassa todos os métodos sem simplificar nada;
- o novo ponto de entrada esconde decisões que o cliente realmente precisa controlar;
- uma única classe passa a concentrar regras de áreas diferentes;
- a camada extra dificulta mais do que ajuda.

## Em uma frase

**Facade Pattern cria uma entrada simples para um subsistema complexo, reduzindo o acoplamento do cliente sem obrigatoriamente remover a complexidade interna.**

### Veja também

- [[Design Pattern]]
- [[Factory Pattern]]
- [[Backend]]
- [[Frontend]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Quarkus]]
- [[Outbox Pattern]]
- [[Chave de idempotência]]
- [[Idempotência]]
- [[Requisição]]
- [[JSON]]
- [[Testes]]
- [[Segurança]]
- [[SOLID]]
