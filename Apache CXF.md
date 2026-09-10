# O que é Apache CXF?

**Apache CXF** é um framework Java para criar, publicar e consumir serviços web. Ele oferece suporte principalmente a serviços SOAP com JAX-WS e a serviços REST com JAX-RS, além de clientes, geração de código, transportes, serialização e interceptors.

Uma analogia: o CXF é como uma central de tradução e despacho. Ele recebe uma mensagem de um sistema, entende o protocolo e o formato, transforma os dados em objetos Java, chama o serviço correto e monta a resposta no formato esperado.

O CXF não é um banco de dados, um servidor web isolado ou uma linguagem. Ele é uma infraestrutura que pode ser colocada dentro de uma aplicação [[Java]] para permitir a comunicação entre sistemas.

## Para que usar?

O Apache CXF costuma ser usado quando uma aplicação precisa:

- expor ou consumir serviços SOAP;
- trabalhar com contratos WSDL;
- integrar sistemas corporativos ou legados;
- usar padrões WS-* como WS-Addressing e WS-Security;
- gerar classes Java a partir de um WSDL;
- criar clientes para serviços externos;
- publicar APIs REST usando JAX-RS;
- controlar detalhes da mensagem com interceptors;
- transmitir anexos binários com MTOM.

Ele é especialmente útil quando um parceiro já exige SOAP, WSDL ou políticas WS-Security. Para uma API nova, simples e baseada em HTTP e [[JSON]], o recurso REST nativo do framework da aplicação pode ser mais direto.

## SOAP e REST no CXF

O CXF pode trabalhar com os dois estilos, mas eles possuem ideias diferentes:

| Característica | SOAP | REST |
|---|---|---|
| Contrato comum | WSDL e schemas XML | Recursos, URLs e contrato de API |
| Mensagem | Envelope SOAP, geralmente XML | HTTP com JSON, XML ou outro formato |
| Operação | Operações definidas no serviço | Verbos HTTP como GET, POST, PUT e DELETE |
| Segurança | Pode usar WS-Security, além de TLS | Normalmente usa TLS e autenticação da API |
| Erros | SOAP Fault | Status HTTP e corpo de erro |
| Uso frequente | Integrações corporativas e contratos rígidos | APIs web e aplicações modernas |

SOAP e REST não são simplesmente “melhor” e “pior”. A escolha depende do contrato do parceiro, dos requisitos de segurança, do ecossistema e do custo de manutenção.

## JAX-WS e SOAP

**JAX-WS** é uma API Java para serviços web baseados em SOAP. No CXF, uma interface Java pode representar as operações oferecidas por um serviço.

Exemplo simplificado:

```java
@WebService
public interface SaudacaoPort {
    String dizerOla(String nome);
}
```

Uma implementação pode ser:

```java
@WebService(endpointInterface = "com.exemplo.SaudacaoPort")
public class SaudacaoService implements SaudacaoPort {

    @Override
    public String dizerOla(String nome) {
        return "Olá, " + nome;
    }
}
```

O CXF cria ou configura o endpoint que recebe a mensagem SOAP, converte o XML para os tipos Java, chama `dizerOla` e transforma o resultado em uma resposta SOAP.

Os imports exatos podem variar conforme a versão e o uso de APIs `javax` ou `jakarta`. Em projetos novos, confirme a combinação de versões do CXF, do JDK e do framework.

## JAX-RS e REST

**JAX-RS** é uma API Java para criar serviços REST. Um recurso pode ser descrito com anotações como `@Path`, `@GET` e `@Produces`:

```java
@Path("/clientes")
public class ClienteResource {

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Cliente> listar() {
        return List.of();
    }
}
```

O CXF usa o modelo JAX-RS para mapear URLs e verbos HTTP para métodos Java. Em uma aplicação [[Backend]], o recurso deve receber a entrada, validar os dados, chamar um service e devolver uma resposta consistente.

O uso de JAX-RS pelo CXF não significa que toda API REST precise dele. [[Quarkus]] possui seu próprio suporte REST; escolha o mecanismo de acordo com o projeto e a integração necessária.

## WSDL e contratos

**WSDL** (*Web Services Description Language*) é um documento que descreve um serviço SOAP: operações, tipos de dados, mensagens, bindings e endpoints.

O WSDL funciona como um contrato entre sistemas. É parecido com um formulário oficial: antes de conversar, o cliente e o servidor concordam sobre os nomes, tipos e formatos aceitos.

Existem duas abordagens comuns:

### Contract first

O WSDL é definido primeiro. A partir dele, ferramentas geram interfaces, classes e objetos auxiliares para Java.

Essa abordagem é útil quando o contrato pertence a um parceiro ou precisa ser controlado com rigor.

### Code first

As classes e anotações Java são criadas primeiro. O framework gera o WSDL a partir delas.

Essa abordagem pode ser mais rápida para começar, mas mudanças no código podem alterar o contrato publicado. Em serviços compartilhados, revise o WSDL antes de publicar uma alteração.

## Geração de código com `wsdl2java`

O CXF possui ferramentas para gerar Java a partir de WSDL. O processo normalmente é:

```text
WSDL + schemas
        ↓ wsdl2java
interfaces, classes de dados e cliente
        ↓
implementação ou chamada do serviço
```

No [[Maven]], o plugin de geração é o `cxf-codegen-plugin`:

```xml
<plugin>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-codegen-plugin</artifactId>
    <version>${cxf.version}</version>
</plugin>
```

O exemplo mostra a identificação do plugin; a configuração completa depende da localização do WSDL, dos schemas, do diretório de saída e das opções de geração.

Boas práticas para código gerado:

- não edite manualmente os arquivos gerados;
- mantenha o WSDL e os schemas versionados;
- gere o código durante o build ou em um processo reproduzível;
- mantenha o código gerado separado do código escrito pela equipe;
- revise mudanças depois de atualizar o contrato;
- confira se o package gerado não conflita com classes da aplicação.

## Dependências Maven

Os módulos dependem do que a aplicação precisa. Exemplos de módulos do CXF incluem frontend JAX-WS, frontend JAX-RS e transporte HTTP:

```xml
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-frontend-jaxws</artifactId>
    <version>${cxf.version}</version>
</dependency>

<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-transports-http</artifactId>
    <version>${cxf.version}</version>
</dependency>
```

Não adicione todos os módulos sem necessidade. Use gerenciamento de dependências compatível e mantenha os módulos na mesma linha de versão para reduzir conflitos.

## Arquitetura do CXF

O CXF é composto por partes que trabalham em conjunto:

- **Bus:** registro central de extensões, propriedades e interceptors;
- **Frontend:** modelo de programação, como JAX-WS ou JAX-RS;
- **Service model:** representação de operações, endpoints e contratos;
- **Data binding:** conversão entre XML ou JSON e objetos Java;
- **Protocol binding:** interpretação do protocolo e do formato da mensagem;
- **Transport:** envio e recebimento por HTTP ou outros transportes;
- **Endpoint:** ponto que representa um serviço publicado ou consumido;
- **Client e Server:** componentes para chamadas e serviços;
- **Interceptor chain:** sequência de etapas que processa a mensagem.

Separar essas partes permite adaptar o CXF a diferentes protocolos, formatos e formas de implantação.

## Interceptors

Um **interceptor** é uma etapa que observa ou altera uma mensagem durante o processamento.

Uma analogia é uma sequência de portões em um aeroporto:

1. um portão lê a identificação;
2. outro confere a bagagem;
3. outro valida o destino;
4. o passageiro chega ao voo.

No CXF, interceptors podem:

- ler headers;
- validar XML e schemas;
- converter dados;
- registrar métricas;
- aplicar autenticação;
- adicionar headers;
- tratar falhas;
- fazer logging;
- aplicar políticas de segurança.

Existem cadeias de entrada, saída e falha, tanto no cliente quanto no servidor. Um interceptor instalado no lugar errado pode não executar quando esperado.

## Segurança

O CXF possui suporte a mecanismos como WS-Security e pode usar interceptors para assinaturas, criptografia, tokens e políticas SOAP. Isso não significa que uma aplicação esteja segura por padrão.

Boas práticas:

- use [[TLS]] para proteger o transporte;
- valide certificados e nomes de host;
- configure WS-Security conforme o contrato do parceiro;
- prefira credenciais com menor privilégio;
- valide XML e limite tamanho de mensagens e anexos;
- proteja contra XML External Entity e outras ameaças de parser;
- defina timeouts de conexão e leitura;
- não registre XML completo quando houver senhas, tokens ou dados pessoais;
- mascare headers e elementos sensíveis nos logs;
- mantenha o CXF e as dependências atualizados;
- teste falhas, autenticação, autorização e replay quando aplicável.

O guia oficial do CXF descreve o uso de interceptors WSS4J para configurar WS-Security. As escolhas de segurança devem ser combinadas com as regras gerais de [[Segurança]] da aplicação.

## Logging e dados sensíveis

O logging de mensagens é útil para investigar integrações, mas uma mensagem SOAP pode conter documentos inteiros, credenciais, identificadores e dados pessoais.

Antes de habilitar logs em produção:

- defina quais headers e elementos devem ser mascarados;
- desabilite o corpo quando não for necessário;
- controle acesso aos arquivos de log;
- defina retenção e eliminação;
- evite logs em nível muito detalhado durante todo o tempo;
- reproduza problemas em um ambiente seguro quando possível.

Um log que resolve um incidente, mas expõe senhas, cria um novo incidente de segurança.

## Cliente e servidor

O CXF pode atuar dos dois lados:

```text
Aplicação A
  cliente CXF -> mensagem SOAP/REST -> servidor CXF da Aplicação B
```

No lado cliente, ele prepara a mensagem, aplica interceptors, envia pelo transporte e converte a resposta.

No lado servidor, ele recebe a requisição, processa a cadeia de entrada, encontra a operação, chama a implementação e monta a resposta.

Uma falha pode acontecer em qualquer etapa. Por isso, diferencie:

- erro de DNS ou conexão;
- timeout;
- falha TLS;
- erro de autenticação;
- SOAP Fault;
- status HTTP de erro;
- erro de conversão XML ou JSON;
- exceção na regra de negócio.

## CXF com Quarkus

O projeto **Quarkus CXF** é uma extensão do Quarkiverse que integra as bibliotecas Apache CXF ao Quarkus, principalmente para clientes SOAP e serviços JAX-WS.

Uma dependência básica da extensão é:

```xml
<dependency>
    <groupId>io.quarkiverse.cxf</groupId>
    <artifactId>quarkus-cxf</artifactId>
</dependency>
```

O gerenciamento da versão deve seguir a documentação e o BOM compatível com a versão do Quarkus. Não copie uma versão antiga sem conferir a matriz de compatibilidade.

A extensão oferece recursos para:

- publicar serviços SOAP;
- criar clientes SOAP;
- gerar Java a partir de WSDL;
- gerar WSDL a partir de Java;
- configurar interceptors e features;
- configurar TLS;
- empacotar para JVM e nativo;
- trabalhar com CDI e configuração do Quarkus.

Apache CXF pode ser usado diretamente em Java, enquanto Quarkus CXF acrescenta integração com o modelo e o ciclo de build do Quarkus. São camadas relacionadas, mas não são o mesmo projeto.

## CXF e CDI

Em uma aplicação Quarkus, a implementação do serviço pode ser integrada ao CDI. Isso permite injetar repositórios, clientes e serviços, em vez de criar tudo manualmente.

```java
@ApplicationScoped
public class SaudacaoService implements SaudacaoPort {

    private final AuditoriaService auditoria;

    @Inject
    public SaudacaoService(AuditoriaService auditoria) {
        this.auditoria = auditoria;
    }

    @Override
    public String dizerOla(String nome) {
        auditoria.registrar(nome);
        return "Olá, " + nome;
    }
}
```

O serviço SOAP continua sendo exposto pelo CXF, mas suas dependências internas podem ser gerenciadas pelo container. Consulte também [[Chamada estática vs injeção CDI]] para entender a diferença entre criar e chamar serviços diretamente e usar injeção.

## REST com CXF ou REST do Quarkus?

Para uma API REST nova no projeto, compare as opções:

- **REST do Quarkus:** normalmente é a opção natural quando o projeto já usa Quarkus e precisa de endpoints REST;
- **JAX-RS do CXF:** pode fazer sentido quando a equipe já usa CXF, precisa de recursos específicos ou quer manter uma integração existente;
- **CXF para SOAP:** é a escolha natural quando o contrato externo exige SOAP, WSDL ou WS-*.

Não adicione CXF somente porque ele é capaz de fazer REST. Considere dependências, curva de aprendizado, compatibilidade, suporte da equipe e o contrato que precisa ser atendido.

## Testes

Uma integração CXF deve ser testada em camadas:

- teste unitário das regras de negócio sem rede;
- teste do mapeamento da interface para o contrato;
- teste de serialização e desserialização;
- teste do WSDL e dos schemas;
- teste de interceptors e políticas de segurança;
- teste de cliente com um servidor controlado;
- teste de timeout, falha, SOAP Fault e status HTTP;
- teste de compatibilidade com o sistema parceiro.

Use [[Testes]] de contrato quando o WSDL ou o formato da mensagem for parte importante da integração. Testes de integração ajudam a encontrar problemas que não aparecem em testes unitários, como namespaces, headers e configuração TLS.

## Boas práticas

1. **Trate o WSDL como contrato.** Versione, revise e teste mudanças antes de publicar.
2. **Use as dependências mínimas.** Evite carregar módulos do CXF sem necessidade.
3. **Mantenha as versões alinhadas.** CXF, plugins, JAXB, JAX-WS e o framework precisam ser compatíveis.
4. **Separe código gerado e código manual.** Nunca dependa de alterações manuais em arquivos que serão regenerados.
5. **Defina timeouts.** Uma chamada externa sem timeout pode prender threads e recursos.
6. **Trate retries com cuidado.** Repetir uma operação de escrita pode duplicar efeitos; considere [[Idempotência]].
7. **Proteja os logs.** XML e JSON podem conter informações sensíveis.
8. **Valide mensagens.** Use schemas e validações de negócio, não apenas conversão de tipos.
9. **Use TLS corretamente.** Não desabilite a validação de certificado para “resolver” um problema de ambiente.
10. **Monitore falhas por etapa.** Diferencie transporte, protocolo, autenticação, conversão e regra de negócio.
11. **Teste o parceiro real quando possível.** Mocks não reproduzem todos os detalhes de WS-* e do contrato.
12. **Não confunda SOAP Fault com exceção genérica.** Mapeie erros para o contrato esperado pelo consumidor.
13. **Prefira CDI para colaboradores da aplicação.** Deixe funções puras como utilitários quando fizer sentido.
14. **Acompanhe avisos de segurança.** Frameworks de integração processam XML, headers, anexos e dados externos.

## Resumo

Apache CXF é um framework Java para criar e consumir serviços web, com suporte forte a SOAP, WSDL, JAX-WS, REST com JAX-RS e interceptors. Ele é uma boa escolha para integrações corporativas e contratos SOAP, especialmente quando há requisitos de WS-* ou um parceiro legado. Em projetos Quarkus, a extensão Quarkus CXF integra o CXF ao framework. Para qualquer uso, cuide de versões, contratos, timeouts, logs, TLS, validação, segurança e testes.

### Veja também

- [[Java]]
- [[Maven]]
- [[Quarkus]]
- [[Backend]]
- [[Framework]]
- [[JSON]]
- [[Requisição]]
- [[TLS]]
- [[Segurança]]
- [[Testes]]
- [[Idempotência]]
- [[Caddy]]
- [[Chamada estática vs injeção CDI]]

### Referências oficiais

- [Apache CXF — arquitetura](https://cxf.apache.org/docs/cxf-architecture.html)
- [Apache CXF — JAX-RS](https://cxf.apache.org/docs/jax-rs.html)
- [Apache CXF — interceptors](https://cxf.apache.org/docs/interceptors.html)
- [Apache CXF — WS-Security](https://cxf.apache.org/docs/ws-security.html)
- [Quarkus CXF — documentação](https://docs.quarkiverse.io/quarkus-cxf/dev/reference/extensions/quarkus-cxf.html)
