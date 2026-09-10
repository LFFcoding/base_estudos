# Chamada estática vs injeção CDI

Uma **chamada estática** usa um método ou campo marcado com `static` e pertence à classe, não a uma instância específica.

**Injeção CDI** (*Contexts and Dependency Injection*) entrega uma dependência para um objeto gerenciado pelo container. Em vez de a classe criar ou localizar tudo sozinha, ela declara do que precisa e o CDI monta as conexões.

Uma analogia:

- chamada estática é ligar diretamente para um número fixo de uma agenda;
- injeção CDI é pedir um serviço a uma recepção, que entrega a implementação correta conforme as regras do prédio.

As duas abordagens podem executar código, mas têm objetivos diferentes.

## Chamada estática

```java
public final class TextoUtil {

    private TextoUtil() {
    }

    public static String normalizar(String texto) {
        return texto == null ? "" : texto.trim().toLowerCase();
    }
}
```

Uso:

```java
String resultado = TextoUtil.normalizar("  Java  ");
```

Nesse caso, não é necessário criar um objeto `TextoUtil`. O método recebe uma entrada, calcula um resultado e não depende de banco, configuração, rede ou outro serviço.

## Injeção CDI

Com CDI, uma classe pode declarar uma dependência no construtor:

```java
@ApplicationScoped
public class PedidoService {

    private final Notificador notificador;

    @Inject
    public PedidoService(Notificador notificador) {
        this.notificador = notificador;
    }

    public void aprovarPedido(Long pedidoId) {
        // Regras de aprovação...
        notificador.enviar("Pedido aprovado: " + pedidoId);
    }
}
```

Uma implementação pode ser registrada como bean:

```java
@ApplicationScoped
public class EmailNotificador implements Notificador {

    @Override
    public void enviar(String mensagem) {
        // Envia e-mail por um cliente configurado.
    }
}
```

O container CDI descobre os beans, resolve a implementação compatível e injeta a dependência no `PedidoService`.

Em [[Quarkus]], a solução de injeção é chamada **ArC** e é baseada na especificação Jakarta CDI. O framework cria, conecta e destrói beans de acordo com o ciclo de vida e o escopo configurados.

## Comparação rápida

| Característica | Chamada estática | Injeção CDI |
|---|---|---|
| Criação | Não há instância da classe utilitária. | O container cria ou administra o bean. |
| Dependências | Geralmente ficam ocultas ou são acessadas diretamente. | São declaradas explicitamente. |
| Estado | Frequentemente global quando existe estado estático. | Controlado pelo escopo do bean. |
| Substituição | Mais difícil trocar a implementação. | Pode usar interfaces, qualifiers e alternativas. |
| Testes | Pode exigir ferramentas específicas ou estado global. | É simples passar um fake ou mock pelo construtor. |
| Configuração | Precisa ser buscada de forma manual. | Pode ser injetada e gerenciada pelo framework. |
| Ciclo de vida | A JVM controla membros estáticos. | O CDI controla o ciclo de vida do bean. |
| Interceptores | Não são aplicados como em uma chamada de bean. | Pode participar de interceptores, eventos e outros recursos CDI. |

## O container CDI

O **container** é a parte do runtime que administra os beans. Ele pode:

- descobrir classes elegíveis;
- criar e destruir instâncias;
- resolver dependências;
- aplicar escopos;
- executar callbacks de ciclo de vida;
- aplicar interceptores;
- selecionar implementações por qualifiers;
- disparar e observar eventos.

Por isso, uma classe gerenciada pelo CDI é mais do que uma classe criada com `new`. Ela participa de um ambiente que conhece suas dependências e seu ciclo de vida.

## Escopos importantes

O escopo define por quanto tempo uma instância existe e em quais contextos ela pode ser compartilhada.

### `@ApplicationScoped`

Normalmente representa uma instância compartilhada pela aplicação. É adequado para serviços sem estado específico de uma requisição, desde que sejam seguros para uso concorrente.

```java
@ApplicationScoped
public class CalculadoraService {
}
```

### `@RequestScoped`

Relaciona a instância ao ciclo de uma requisição HTTP. É útil quando o estado pertence somente àquela requisição.

```java
@RequestScoped
public class ContextoDaRequisicao {
}
```

### `@Dependent`

É um pseudoescopo. A instância acompanha o objeto ou a operação que a utiliza, em vez de ser compartilhada como um bean normal.

### `@Singleton`

Também representa uma instância única, mas possui diferenças de ciclo de vida e proxy em relação a `@ApplicationScoped`. Não escolha somente pelo nome: leia as características do escopo e verifique se o estado é seguro para múltiplas requisições.

Use o escopo adequado. Um bean `@ApplicationScoped` com campos mutáveis específicos de cada usuário pode causar vazamento de dados entre requisições.

## Por que não usar tudo como `static`?

Métodos estáticos são úteis, mas serviços da aplicação frequentemente precisam de:

- configuração de ambiente;
- clientes HTTP;
- conexão com banco;
- [[ORM]];
- logs;
- cache;
- fila ou outro serviço externo;
- implementação substituível;
- controle de ciclo de vida.

Se tudo for acessado por métodos estáticos, essas dependências podem ficar escondidas no código. A classe parece simples, mas depende de estado global ou de serviços que não aparecem em seu construtor.

Exemplo difícil de testar:

```java
public class PedidoService {

    public void aprovar(Long pedidoId) {
        EmailUtil.estaticoEnviar("Pedido aprovado: " + pedidoId);
        AuditoriaGlobal.registrar(pedidoId);
    }
}
```

O teste precisa lidar com `EmailUtil` e `AuditoriaGlobal` reais, globais ou com uma ferramenta que intercepte chamadas estáticas.

Versão com dependências explícitas:

```java
public class PedidoService {

    private final Notificador notificador;
    private final Auditoria auditoria;

    public PedidoService(Notificador notificador, Auditoria auditoria) {
        this.notificador = notificador;
        this.auditoria = auditoria;
    }

    public void aprovar(Long pedidoId) {
        notificador.enviar("Pedido aprovado: " + pedidoId);
        auditoria.registrar(pedidoId);
    }
}
```

Agora o comportamento fica mais claro e as implementações podem ser substituídas.

## CDI não é uma chamada mágica

O CDI não torna qualquer classe automaticamente injetável. Para a injeção funcionar, normalmente é necessário:

1. incluir a dependência e a extensão corretas no projeto;
2. ter uma classe descoberta pelo CDI;
3. declarar um escopo ou outra forma válida de bean;
4. garantir que exista uma implementação compatível;
5. resolver ambiguidades com qualifiers ou alternativas;
6. usar a instância dentro de um contexto ativo.

Se houver duas implementações da mesma interface sem uma escolha clara, o container pode não saber qual deve injetar. Nesse caso, use qualifiers:

```java
@Qualifier
@Retention(RUNTIME)
public @interface PorEmail {
}
```

```java
@ApplicationScoped
@PorEmail
public class EmailNotificador implements Notificador {
}
```

```java
@Inject
public PedidoService(@PorEmail Notificador notificador) {
    this.notificador = notificador;
}
```

O qualifier funciona como uma etiqueta que informa ao CDI qual bean escolher.

## Injeção no construtor

A injeção pelo construtor costuma ser a opção mais clara porque:

- mostra todas as dependências obrigatórias;
- permite criar a classe manualmente em um teste simples;
- ajuda a manter o objeto válido desde a construção;
- evita campos que podem permanecer `null`.

```java
@ApplicationScoped
public class RelatorioService {

    private final RelatorioRepository repository;

    @Inject
    public RelatorioService(RelatorioRepository repository) {
        this.repository = repository;
    }
}
```

Em versões e situações específicas do Quarkus, o `@Inject` pode ser dispensado em um único construtor. Usá-lo explicitamente pode deixar o exemplo mais portável e mais fácil de reconhecer por quem está aprendendo CDI.

## CDI e chamadas estáticas podem coexistir

Escolher CDI não significa eliminar todo `static`. É normal ter uma classe de função pura:

```java
public final class PrecoUtil {

    private PrecoUtil() {
    }

    public static BigDecimal aplicarDesconto(
            BigDecimal preco,
            BigDecimal percentual) {
        return preco.subtract(preco.multiply(percentual));
    }
}
```

Essa função pode permanecer estática se não tiver dependências, estado global ou efeitos externos.

Também é possível um bean CDI chamar uma função estática pura:

```java
@ApplicationScoped
public class PedidoService {

    public BigDecimal calcularTotal(BigDecimal valor) {
        return PrecoUtil.aplicarDesconto(valor, new BigDecimal("0.10"));
    }
}
```

O problema não é a palavra `static` sozinha. O problema aparece quando ela é usada para esconder serviços, estado compartilhado ou dependências que deveriam ser controladas pelo container.

## CDI e testes

A injeção de dependências facilita os [[Testes]] porque uma implementação falsa pode ser passada ao construtor:

```java
var notificadorFake = new NotificadorFake();
var auditoriaFake = new AuditoriaFake();
var service = new PedidoService(notificadorFake, auditoriaFake);

service.aprovar(42L);
```

O teste verifica o comportamento sem enviar e-mail nem gravar em um sistema externo.

Com métodos estáticos que acessam rede ou banco, esse isolamento costuma ser mais difícil. Isso não torna o teste impossível, mas aumenta o acoplamento e a necessidade de ferramentas específicas.

## CDI, SOLID e Design Patterns

Injeção de dependências apoia vários princípios de [[SOLID]], principalmente:

- **Responsabilidade única:** a classe usa serviços sem assumir a responsabilidade de construí-los;
- **Inversão de dependência:** o serviço pode depender de uma abstração, como uma interface;
- **Aberto/fechado:** novas implementações podem ser adicionadas com menos alterações no consumidor.

Ela também aparece em padrões como **Dependency Injection**, **Strategy**, **Factory** e **Decorator**, explicados em [[Design Pattern]].

## Como escolher?

Use uma chamada estática quando:

- a função é pura e determinística;
- não há dependências externas;
- não existe estado compartilhado mutável;
- a operação é uma utilidade pequena e bem nomeada;
- criar um bean só adicionaria complexidade.

Prefira injeção CDI quando:

- a classe é um serviço da aplicação;
- existem dependências como banco, APIs, filas ou configurações;
- a implementação pode mudar;
- é necessário controlar ciclo de vida e escopo;
- a classe será testada com fakes ou mocks;
- interceptores, eventos ou decorators são úteis;
- o código pertence ao fluxo de um [[Backend]].

Uma regra simples: **funções puras podem ser estáticas; colaboradores da aplicação devem ser dependências explícitas**.

## Boas práticas

1. **Prefira injeção por construtor** para dependências obrigatórias.
2. **Evite estado global mutável** em campos estáticos.
3. **Não use `static` para esconder uma dependência** de banco, rede ou configuração.
4. **Use interfaces quando a implementação puder variar** ou precisar ser substituída em testes.
5. **Escolha o escopo CDI conscientemente** e verifique concorrência e compartilhamento de estado.
6. **Não crie beans para qualquer função pequena.** Uma função pura simples pode continuar sendo uma utility.
7. **Não use `new` dentro do serviço para criar colaboradores gerenciáveis pelo CDI.** Injete-os.
8. **Não tente injetar dependências em campos estáticos.** O CDI gerencia instâncias, não variáveis globais da classe.
9. **Use qualifiers para escolhas explícitas**, evitando ambiguidades no container.
10. **Mantenha segredos e configurações fora do código**, usando os mecanismos adequados do ambiente.
11. **Teste com implementações falsas** antes de depender de mocks complexos.
12. **Documente exceções.** Se uma dependência precisa ser obtida manualmente, explique o motivo.
13. **Considere o acoplamento, não apenas a quantidade de linhas.** Código curto pode ser difícil de alterar se depender de estado global.

## Resumo

Uma chamada estática é adequada para funções simples, puras e sem dependências. A injeção CDI é adequada para serviços e colaboradores que precisam de ciclo de vida, configuração, substituição, interceptores ou integração com o restante da aplicação. Em [[Quarkus]], o CDI ajuda o container a montar essas relações. A escolha deve tornar as dependências visíveis, os testes mais fáceis e o código mais seguro de manter.

### Veja também

- [[Java]]
- [[Quarkus]]
- [[Backend]]
- [[Framework]]
- [[Maven]]
- [[ORM]]
- [[SOLID]]
- [[Design Pattern]]
- [[Testes]]
- [[Segurança]]

### Referências oficiais

- [Introdução ao CDI no Quarkus](https://quarkus.io/guides/cdi)
- [Referência de CDI no Quarkus](https://quarkus.io/guides/cdi-reference)
- [Especificação Jakarta CDI](https://jakarta.ee/specifications/cdi/5.0/jakarta-cdi-spec-5.0)
