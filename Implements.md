# O que significa `implements`?

`implements` é uma palavra-chave usada principalmente em [[Java]] para indicar que uma classe cumpre o contrato de uma interface.

Quando uma classe implementa uma interface, ela promete fornecer as operações definidas por essa interface. A interface diz **o que** pode ser feito; a classe define **como** fazer.

Uma analogia: uma empresa pode exigir que todo funcionário saiba “registrar ponto”. Cada funcionário implementa essa regra usando seu próprio dispositivo ou processo, mas todos precisam oferecer a mesma capacidade.

## Exemplo básico em Java

```java
public interface Notificador {
    void enviar(String mensagem);
}

public final class NotificadorEmail implements Notificador {
    @Override
    public void enviar(String mensagem) {
        System.out.println("Enviando e-mail: " + mensagem);
    }
}
```

Nesse exemplo:

- `Notificador` é a interface;
- `NotificadorEmail` é a classe concreta;
- `implements Notificador` declara que a classe cumpre o contrato;
- `@Override` confirma que `enviar` implementa o método da interface;
- `public` é necessário porque o método da interface é público.

Se a classe concreta não implementar todos os métodos abstratos da interface, o compilador apresentará um erro.

## Usando a implementação pelo contrato

O código consumidor pode depender da interface:

```java
Notificador notificador = new NotificadorEmail();
notificador.enviar("Seu pedido foi enviado");
```

A variável é do tipo `Notificador`, mas o objeto real é `NotificadorEmail`. Essa escolha é um exemplo de [[Polimorfismo]]. O consumidor chama o contrato sem conhecer os detalhes internos da implementação.

## Uma classe pode implementar várias interfaces

Em Java, uma classe pode implementar mais de uma interface:

```java
public final class RelatorioPdf
        implements Exportavel, Auditavel {

    @Override
    public byte[] exportar() {
        return new byte[0];
    }

    @Override
    public void auditar() {
        System.out.println("Relatório exportado");
    }
}
```

Isso permite combinar capacidades diferentes sem criar uma hierarquia de classes profunda. Cada interface deve representar um contrato coeso, e não uma lista aleatória de métodos.

## [[Extends]] versus `implements`

Os dois termos têm funções diferentes:

| Palavra | Uso comum | Significado |
|---|---|---|
| `extends` | classe herda de classe ou interface herda de interface | cria uma relação de extensão |
| `implements` | classe implementa interface | cumpre um contrato |

Exemplo:

```java
public abstract class Documento {
    public abstract String titulo();
}

public interface Exportavel {
    byte[] exportar();
}

public final class Relatorio
        extends Documento
        implements Exportavel {

    @Override
    public String titulo() {
        return "Relatório mensal";
    }

    @Override
    public byte[] exportar() {
        return new byte[0];
    }
}
```

`Relatorio` herda a base de `Documento` e também cumpre o contrato de `Exportavel`.

Em Java, uma classe pode estender uma classe-base, mas pode implementar várias interfaces. Uma interface pode estender outras interfaces usando `extends`; ela não usa `implements`.

## Implementação parcial

Uma classe abstrata pode implementar uma interface sem implementar todos os seus métodos:

```java
public abstract class NotificadorBase implements Notificador {
    // Ainda não implementa enviar.
}
```

Como `NotificadorBase` é abstrata, ela pode deixar `enviar` para uma subclasse concreta. Uma classe concreta, porém, precisa completar todos os métodos obrigatórios, salvo os que já possuem implementação padrão.

## Métodos `default` em interfaces

Uma interface Java pode fornecer um comportamento padrão:

```java
public interface Identificavel {
    String id();

    default boolean possuiId() {
        return id() != null && !id().isBlank();
    }
}
```

Uma classe que implementa `Identificavel` precisa fornecer `id`, mas pode reutilizar `possuiId`. Ela também pode sobrescrever o método padrão se precisar de outro comportamento.

Use métodos `default` com moderação. Eles são úteis para comportamento realmente comum, mas podem colocar regras demais em uma interface e tornar sua evolução confusa.

## `implements` em C#

Em [[C#]], a ideia existe, mas a palavra `implements` não é usada. A interface aparece depois de `:`:

```csharp
public interface INotificador
{
    void Enviar(string mensagem);
}

public sealed class NotificadorEmail : INotificador
{
    public void Enviar(string mensagem)
    {
        Console.WriteLine($"Enviando e-mail: {mensagem}");
    }
}
```

Em C#, a mesma sintaxe lista a classe-base e as interfaces:

```csharp
public class Relatorio : DocumentoBase, IExportavel, IAuditavel
{
    // Implementações dos contratos.
}
```

A classe-base deve aparecer primeiro. As interfaces vêm depois, separadas por vírgula.

Use `override` quando estiver sobrescrevendo um método `virtual` ou `abstract` de uma classe-base. Para membros de interface, a implementação normalmente possui a mesma assinatura pública, ou pode usar implementação explícita quando for necessário resolver conflitos.

## Implementação explícita em C#

Uma classe pode implementar duas interfaces que possuem métodos com o mesmo nome, fornecendo uma implementação diferente para cada contrato:

```csharp
public interface ILeitor
{
    string Ler();
}

public interface IPreview
{
    string Ler();
}

public sealed class Documento : ILeitor, IPreview
{
    string ILeitor.Ler() => "Conteúdo completo";

    string IPreview.Ler() => "Prévia";
}
```

Nesse caso, o método é acessado por meio da interface correspondente. Use esse recurso quando os contratos realmente exigirem comportamentos diferentes; não o use apenas para esconder uma API mal projetada.

## Implementação em C++

Em [[C++]], não existe a palavra-chave `implements`. O conceito costuma ser representado por herança pública e funções virtuais puras:

```cpp
#include <string>

class Notificador
{
public:
    virtual ~Notificador() = default;
    virtual void enviar(const std::string& mensagem) = 0;
};

class NotificadorEmail final : public Notificador
{
public:
    void enviar(const std::string& mensagem) override
    {
        // Envia a mensagem por e-mail.
    }
};
```

O `= 0` torna `enviar` uma função virtual pura e faz de `Notificador` uma classe abstrata. `override` informa ao compilador que o método deve sobrescrever um método virtual da base.

Em C++, uma classe que representa uma interface geralmente contém somente operações virtuais públicas e um destrutor virtual. Ela também pode conter dados ou implementação comum, mas isso a aproxima de uma classe abstrata completa.

## `implements` e classe abstrata

Uma classe abstrata pode implementar uma interface e deixar parte do contrato para suas subclasses. A relação é:

```text
Interface: define o contrato
        ↓ implements
Classe abstrata: compartilha parte da implementação
        ↓ extends
Classe concreta: completa o comportamento
```

Por exemplo:

```java
public interface Persistivel {
    void salvar();
}

public abstract class EntidadeBase implements Persistivel {
    protected long id;

    public long id() {
        return id;
    }
}

public final class Pedido extends EntidadeBase {
    @Override
    public void salvar() {
        System.out.println("Salvando pedido");
    }
}
```

`EntidadeBase` fornece estado comum, e `Pedido` completa a operação `salvar`.

## Relação com interfaces abstratas

O arquivo [[Interface abstrata]] explica por que interfaces já representam contratos abstratos. `implements` é a declaração feita pela classe que assume esse contrato.

Não confunda:

- **interface:** o contrato ou capacidade;
- **implements:** a declaração de que uma classe cumpre o contrato;
- **abstract class:** uma base que não pode ser criada diretamente;
- **extends:** a relação de herança ou extensão.

## Uso no backend

Em um [[Backend]], `implements` aparece em integrações e regras substituíveis:

```java
public interface GatewayPagamento {
    ResultadoPagamento cobrar(BigDecimal valor);
}

public final class GatewayPix implements GatewayPagamento {
    @Override
    public ResultadoPagamento cobrar(BigDecimal valor) {
        return ResultadoPagamento.aprovado();
    }
}
```

O serviço pode receber `GatewayPagamento` por injeção e não precisa conhecer a classe `GatewayPix`. Isso facilita trocar o provedor e testar o fluxo com um fake.

Essa estrutura aparece com [[Strategy Pattern]], [[Factory Pattern]] e injeção de dependência. A factory pode escolher a implementação; a interface define o contrato; o serviço usa a abstração.

## Testes

Interfaces permitem criar implementações simples para testes:

```java
GatewayPagamento gatewayFake = valor ->
        ResultadoPagamento.aprovado();

var service = new PagamentoService(gatewayFake);
```

Teste tanto a classe consumidora quanto as implementações reais:

- verifique se cada implementação respeita o contrato;
- teste entradas válidas e inválidas;
- teste falhas externas e timeouts;
- confirme que o serviço não depende de detalhes concretos;
- use testes de integração para validar o provedor real quando necessário.

O uso de `implements` não garante qualidade por si só. Um contrato mal definido continua produzindo implementações difíceis de manter.

## Boas práticas

- use interfaces para contratos e capacidades reais;
- mantenha cada interface pequena e coesa;
- escolha nomes ligados ao papel, como `GatewayPagamento` ou `Notificador`;
- use `@Override` em Java e `override` em C++;
- não crie uma interface para cada classe sem haver variação ou fronteira real;
- mantenha as implementações substituíveis;
- respeite as pré-condições e os resultados prometidos pelo contrato;
- prefira composição quando comportamentos precisarem ser combinados;
- use [[SOLID]], especialmente os princípios de substituição, inversão de dependência e segregação de interfaces;
- teste o contrato e cada implementação importante;
- documente efeitos colaterais, erros e limites;
- não confunda uma interface com autenticação, autorização ou [[Segurança]].

## Em uma frase

**`implements` declara que uma classe assume o contrato de uma interface e fornece as implementações necessárias para suas operações.**

### Veja também

- [[Interface abstrata]]
- [[Classe abstrata]]
- [[Polimorfismo]]
- [[Design Pattern]]
- [[Strategy Pattern]]
- [[Factory Pattern]]
- [[SOLID]]
- [[Java]]
- [[C#]]
- [[C++]]
- [[Backend]]
- [[Testes]]
- [[Segurança]]
