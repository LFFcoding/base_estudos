# NestJS

**NestJS** é um framework para criar aplicações de backend com [[Node.js]]. Ele é construído com [[TypeScript]], também permite [[JavaScript]] e oferece uma estrutura organizada para criar APIs, serviços web e aplicações distribuídas.

Uma analogia simples: [[Node.js]] é o motor; bibliotecas como Express ou Fastify fornecem peças para o carro; NestJS é uma planta de montagem que organiza essas peças em módulos, controllers e services.

NestJS ajuda a criar aplicações que recebem [[Requisição|requisições]], aplicam regras de negócio e devolvem respostas, normalmente em [[JSON]]. Ele pode ser usado para construir APIs REST, GraphQL, WebSockets e aplicações de mensageria.

## NestJS não é [[Node.js]]

Eles estão relacionados, mas não são a mesma coisa:

- **[[Node.js]]:** ambiente que executa JavaScript fora do navegador;
- **NestJS:** framework que organiza uma aplicação executada no [[Node.js]];
- **[[TypeScript]]:** linguagem usada frequentemente para escrever o código NestJS, adicionando tipos ao [[JavaScript]];
- **Express ou Fastify:** plataformas HTTP que podem ficar por baixo do NestJS.

Também não é um framework de [[Frontend]]. O NestJS normalmente fica no [[Backend]], enquanto [[Next.js]] e [[React]] podem participar da construção da interface ou de aplicações web no lado do cliente.

## Por que o NestJS existe?

É possível criar um servidor [[Node.js]] diretamente ou usar bibliotecas menores. Em projetos pequenos, isso pode ser suficiente. À medida que a aplicação cresce, porém, algumas perguntas aparecem:

- onde ficam as rotas?
- onde ficam as regras de negócio?
- como as dependências são criadas e compartilhadas?
- como validar dados recebidos?
- como proteger rotas?
- como testar cada parte?
- como evitar que todos os arquivos dependam uns dos outros?

O NestJS oferece convenções e ferramentas para responder a essas perguntas de forma consistente. Ele não elimina a necessidade de pensar na arquitetura, mas fornece uma base comum para a equipe.

## Arquitetura principal

### Módulos (*modules*)

Um módulo agrupa funcionalidades relacionadas. Um `UsuariosModule`, por exemplo, pode reunir controller, service, DTOs e regras relacionadas a usuários.

Os módulos funcionam como caixas organizadoras. Eles podem importar outros módulos e exportar providers que serão usados por outras partes da aplicação.

```typescript
import { Module } from '@nestjs/common';
import { UsuariosController } from './usuarios.controller';
import { UsuariosService } from './usuarios.service';

@Module({
  controllers: [UsuariosController],
  providers: [UsuariosService],
})
export class UsuariosModule {}
```

### Controllers

Controllers recebem requisições HTTP e devolvem respostas. Eles devem cuidar da entrada e da saída, delegando a regra de negócio para providers ou services.

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('usuarios')
export class UsuariosController {
  @Get()
  listar() {
    return [{ id: 1, nome: 'Ana' }];
  }
}
```

Esse controller responde a `GET /usuarios`. O decorator `@Controller('usuarios')` define o prefixo da rota e `@Get()` define o método HTTP.

Um controller não deve concentrar consultas ao banco, regras complexas e integrações externas. Quanto mais fina for essa camada, mais fácil será testar e manter a aplicação.

### Providers e services

Providers são classes que o NestJS consegue criar e entregar a outras classes. Services são um tipo comum de provider e normalmente concentram regras de negócio, consultas e operações reutilizáveis.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsuariosService {
  listar() {
    return [{ id: 1, nome: 'Ana' }];
  }
}
```

O controller pode receber o service pelo construtor:

```typescript
import { Controller, Get } from '@nestjs/common';
import { UsuariosService } from './usuarios.service';

@Controller('usuarios')
export class UsuariosController {
  constructor(private readonly usuariosService: UsuariosService) {}

  @Get()
  listar() {
    return this.usuariosService.listar();
  }
}
```

### Injeção de dependências (*dependency injection*)

Nesse exemplo, o controller não cria o `UsuariosService` com `new`. O container de Inversão de Controle (*IoC container*) do NestJS cria o objeto e o injeta no construtor.

É como pedir uma ferramenta ao almoxarifado em vez de construir uma ferramenta nova dentro de cada sala. Isso facilita trocar implementações, compartilhar recursos e escrever [[Testes]].

A documentação oficial recomenda separar controllers, que lidam com requisições, dos providers, que executam tarefas mais complexas. Ela também relaciona esse desenho aos princípios [[SOLID]].

## Peças que atravessam uma requisição

Além de módulos, controllers e providers, o NestJS oferece componentes para etapas específicas:

- **Middleware:** executa antes do handler e pode observar ou alterar a requisição;
- **Pipes:** transformam e validam dados recebidos;
- **Guards:** decidem se uma requisição pode continuar, sendo úteis para autenticação e autorização;
- **Interceptors:** executam lógica antes e depois do handler, podendo medir tempo ou transformar respostas;
- **Exception filters:** padronizam o tratamento de erros;
- **Decorators:** adicionam metadados e simplificam a configuração de classes e métodos.

Uma visão simplificada do fluxo é:

```text
requisição
    -> middleware
    -> guard
    -> pipe
    -> controller
    -> provider/service
    -> resposta ou exception filter
```

Esses componentes devem ter responsabilidades claras. Por exemplo, um guard decide acesso; ele não deve conter toda a regra de cálculo de preço de um pedido.

## Validação de entrada

Dados vindos do cliente não devem ser aceitos automaticamente. Uma aplicação NestJS pode usar DTOs (*Data Transfer Objects*) e `ValidationPipe` para verificar formato, tipos e campos permitidos.

Exemplo de DTO:

```typescript
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CriarUsuarioDto {
  @IsString()
  @MinLength(2)
  nome!: string;

  @IsEmail()
  email!: string;
}
```

Configuração global:

```typescript
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }),
);
```

- `whitelist` remove propriedades que não foram declaradas;
- `forbidNonWhitelisted` rejeita propriedades inesperadas;
- `transform` permite transformar valores conforme as regras configuradas.

Validação não substitui autorização. Um dado pode estar bem formatado e ainda assim o usuário não ter permissão para alterá-lo. Esses cuidados fazem parte de [[Segurança]].

## Criando um projeto

O CLI (*Command-Line Interface*) oficial pode criar a estrutura inicial:

```bash
npm install -g @nestjs/cli
nest new api-estudos
cd api-estudos
npm run start:dev
```

O primeiro comando instala o CLI; `nest new` cria o projeto; e `start:dev` executa o servidor em modo de desenvolvimento, normalmente com recarga automática.

Para gerar partes da aplicação:

```bash
nest g module usuarios
nest g controller usuarios
nest g service usuarios
```

O prefixo `g` significa `generate`. Gerar arquivos pelo CLI ajuda a seguir o padrão de nomes e a evitar configurações manuais repetitivas.

Depois que o endpoint existir, uma chamada simples pode ser feita com [[cURL]]:

```bash
curl http://localhost:3000/usuarios
```

Esse comando envia uma requisição `GET` para o endpoint local e mostra a resposta no terminal.

## Banco de dados

O NestJS não é um banco de dados. Ele pode se conectar a soluções diferentes por meio de bibliotecas, módulos e clients:

- [[PostgreSQL]] para dados relacionais e transações;
- [[MongoDB]] para documentos;
- [[Prisma]] como uma opção de ORM e cliente tipado no ecossistema [[Node.js]] e [[TypeScript]];
- Redis para cache, sessões ou outros usos, conforme a arquitetura;
- outros bancos conforme o driver e a integração escolhidos.

Uma boa separação mantém o controller distante dos detalhes do driver. O controller chama um service; o service usa um repository ou adapter; o adapter conversa com o banco.

```text
controller -> service -> repository/adapter -> banco de dados
```

O NestJS pode organizar o código, mas não decide sozinho qual banco é correto nem garante que as consultas estejam bem modeladas.

## APIs, mensageria e aplicações maiores

Embora seja muito usado para APIs HTTP, o NestJS também possui recursos para GraphQL, WebSockets, eventos e microservices. A aplicação pode conversar com filas e brokers, como [[RabbitMQ]], usando a integração adequada.

Isso não significa que toda aplicação precise ser dividida em microservices. Comece com uma estrutura simples e extraia componentes somente quando houver uma necessidade real de escala, autonomia, isolamento ou organização.

## NestJS e o backend deste projeto

A stack principal deste repositório usa [[Java|Java 17]] com [[Quarkus]] para o backend. NestJS é uma alternativa baseada em [[Node.js]] e [[TypeScript]]; ele não substitui automaticamente o Quarkus.

| Característica | NestJS | Quarkus |
| --- | --- | --- |
| Linguagem principal | [[TypeScript]]/[[JavaScript]] | [[Java|Java 17]] |
| Ambiente | [[Node.js]] | JVM e modos de execução do Quarkus |
| Organização | módulos, controllers e providers | recursos, serviços, injeção e extensões |
| Uso comum | APIs e serviços em [[Node.js]] | APIs e serviços Java |
| Gerenciamento | npm ou outro gerenciador [[JavaScript]] | [[Maven]] |

As ideias de controller, service, validação, autenticação e testes aparecem nos dois ecossistemas, mas os detalhes, bibliotecas e ferramentas são diferentes.

## Vantagens

- estrutura clara para projetos [[Node.js]] grandes;
- suporte forte a [[TypeScript]];
- injeção de dependências integrada;
- CLI para criar componentes;
- recursos para REST, GraphQL, WebSockets e mensageria;
- organização que facilita testes e colaboração;
- possibilidade de usar Express ou Fastify conforme a configuração.

## Cuidados e limitações

- mais abstrações e convenções para aprender;
- decorators e configuração podem parecer mágicos no começo;
- abstrair demais o framework pode dificultar entender o que ocorre na rede ou no banco;
- usar muitos módulos sem uma divisão clara pode criar dependências circulares;
- adicionar microservices, filas e integrações aumenta a complexidade operacional;
- a aplicação ainda precisa tratar segurança, observabilidade, performance e custos.

Escolha NestJS quando a equipe quiser uma estrutura organizada para [[Node.js]] e [[TypeScript]]. Para um serviço pequeno, Express ou Fastify diretamente podem ser suficientes; para um serviço Java alinhado à stack deste projeto, [[Quarkus]] continua sendo a opção estudada.

## Boas práticas

- mantenha controllers finos e coloque regras de negócio em services;
- organize módulos por domínio, não apenas por tipo de arquivo;
- valide entradas com DTOs e pipes;
- use guards para autenticação e autorização;
- não coloque segredos em código ou no Git;
- trate erros com respostas consistentes e sem vazar informações sensíveis;
- use injeção de dependências para facilitar substituição e testes;
- mantenha acesso a banco em repositories ou adapters;
- configure timeouts, logs e métricas para integrações externas;
- teste regras de negócio, controllers e integrações importantes;
- mantenha dependências atualizadas com revisão de compatibilidade;
- documente a API e versiona contratos quando houver clientes externos.

## Referências oficiais

- [Introdução ao NestJS](https://docs.nestjs.com/first-steps);
- [Controllers](https://docs.nestjs.com/controllers);
- [Providers e injeção de dependências](https://docs.nestjs.com/providers);
- [Modules](https://docs.nestjs.com/modules);
- [Testing](https://docs.nestjs.com/fundamentals/testing).

**NestJS é um framework de backend para [[Node.js]] que organiza aplicações [[TypeScript]] em módulos, controllers e providers, oferecendo injeção de dependências, validação, segurança e integrações para construir serviços mais estruturados.**
