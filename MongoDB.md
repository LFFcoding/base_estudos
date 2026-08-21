# MongoDB

**MongoDB** é um banco de dados [[NoSQL]] orientado a documentos (*document database*). Em vez de organizar os dados principalmente em tabelas e linhas, ele guarda documentos agrupados em coleções.

Uma forma simples de imaginar o MongoDB é como um armário de fichas: cada ficha contém as informações completas de um item, e fichas parecidas ficam na mesma gaveta. Essa estrutura é flexível, mas ainda precisa de regras para evitar dados incompletos ou inconsistentes.

## Conceitos principais

- **Documento (*document*):** conjunto de campos e valores que representa um objeto, como um usuário ou pedido.
- **BSON:** formato binário usado pelo MongoDB para armazenar documentos. Ele se parece com [[JSON]], mas possui tipos adicionais, como datas e identificadores próprios.
- **Coleção (*collection*):** grupo de documentos relacionados. É parecida com uma tabela, mas não é exatamente igual.
- **Banco (*database*):** conjunto de coleções.
- **`_id`:** identificador único de cada documento. O MongoDB cria um automaticamente quando ele não é informado.
- **Consulta (*query*):** filtro usado para encontrar documentos.
- **Índice (*index*):** estrutura que acelera buscas em determinados campos.
- **Agregação (*aggregation*):** sequência de etapas para filtrar, agrupar e transformar documentos.

## Exemplo de documento

```json
{
  "_id": "usuario-42",
  "nome": "Ana",
  "email": "ana@example.com",
  "enderecos": [
    {
      "cidade": "São Paulo",
      "principal": true
    }
  ],
  "ativo": true
}
```

Nesse exemplo, `enderecos` é uma lista e cada endereço é um documento aninhado. Esse tipo de organização é útil quando os endereços normalmente são lidos junto com o usuário.

A flexibilidade não significa que cada documento deve ter um formato aleatório. A aplicação deve definir quais campos são obrigatórios, quais tipos são permitidos e como documentos antigos serão atualizados.

## Operações básicas

Os exemplos abaixo podem ser executados no `mongosh`, o terminal do MongoDB:

```javascript
use estudos

db.usuarios.insertOne({
  nome: "Ana",
  email: "ana@example.com",
  ativo: true
})

db.usuarios.find({ ativo: true })

db.usuarios.updateOne(
  { email: "ana@example.com" },
  { $set: { ativo: false } }
)

db.usuarios.deleteOne({ email: "ana@example.com" })
```

- `insertOne` insere um documento;
- `find` consulta documentos que correspondem ao filtro;
- `updateOne` altera o primeiro documento encontrado;
- `deleteOne` remove o primeiro documento encontrado.

Em uma aplicação real, os filtros de atualização e exclusão devem ser específicos. Um filtro incompleto pode alterar ou remover o documento errado.

Uma consulta pode combinar operadores:

```javascript
db.pedidos.find({
  total: { $gte: 100 },
  status: { $in: ["novo", "pago"] }
})
```

Essa consulta procura pedidos com valor maior ou igual a 100 e com status `novo` ou `pago`.

## Índices

Um índice funciona como o índice de um livro: permite encontrar informações sem examinar todas as páginas. Por exemplo:

```javascript
db.usuarios.createIndex(
  { email: 1 },
  { unique: true }
)
```

Esse índice acelera buscas por e-mail e impede dois usuários de usarem o mesmo endereço. Índices ocupam espaço e tornam algumas escritas mais custosas, portanto devem existir para consultas reais e frequentes.

## Agregações

Uma agregação é uma sequência de etapas. Por exemplo, para somar pedidos pagos por usuário:

```javascript
db.pedidos.aggregate([
  { $match: { status: "pago" } },
  {
    $group: {
      _id: "$usuarioId",
      total: { $sum: "$valor" }
    }
  }
])
```

O `$match` filtra os pedidos pagos e o `$group` agrupa os resultados por usuário, somando o campo `valor`.

## Como modelar os dados

No MongoDB, o formato deve ser pensado a partir das consultas mais importantes:

- **Incorpore (*embed*)** dados que normalmente são lidos juntos, que possuem tamanho limitado e dependem do documento principal.
- **Use referências** quando os dados são compartilhados, independentes ou podem crescer muito.
- **Duplique dados com intenção**, quando isso diminuir consultas, mas defina como as cópias serão atualizadas.
- **Defina validações**, mesmo que o banco permita documentos com campos diferentes.
- **Evite documentos excessivamente grandes**, pois isso dificulta leitura, atualização e manutenção.

O MongoDB oferece transações, mas uma transação não deve ser usada para compensar um modelo de dados mal planejado. Primeiro modele os dados para as operações mais comuns; use transações quando uma alteração realmente precisar ser atômica em vários documentos.

## MongoDB e PostgreSQL

O [[PostgreSQL]] organiza dados principalmente em tabelas relacionais e usa SQL. O MongoDB organiza dados em documentos BSON dentro de coleções e usa sua própria API de consultas.

| Situação | Tendência adequada |
| --- | --- |
| Relacionamentos fortes, regras entre várias entidades e consultas relacionais | [[PostgreSQL]] |
| Dados naturalmente agrupados em documentos e com formato que muda | MongoDB |
| Muitas consultas por relações entre entidades | geralmente [[PostgreSQL]] |
| Catálogos ou eventos com atributos variados | MongoDB pode ser adequado |

Essa comparação não significa que MongoDB seja sempre mais rápido. A escolha deve considerar o padrão de acesso, a consistência, a escala, o custo e o conhecimento da equipe. MongoDB é um exemplo prático do modelo de documentos explicado em [[NoSQL]].

## Uso com uma aplicação

O MongoDB não deve ser acessado diretamente pelo navegador. Um fluxo comum é: **[[Frontend]] → [[Requisição]] → [[Backend]] → MongoDB → [[JSON]]**.

O [[Backend]] autentica o usuário, valida os dados, executa a consulta e devolve apenas as informações que o cliente precisa. Em uma aplicação Java com [[Quarkus]], é possível usar um cliente ou uma extensão compatível com MongoDB, embora o banco principal da stack de estudos seja o [[PostgreSQL]].

## MongoDB local com Docker Compose

Para estudar localmente, o MongoDB pode ser iniciado com [[Docker Compose]]:

```yaml
services:
  mongodb:
    image: mongo:<versao-fixada>
    ports:
      - "127.0.0.1:27017:27017"
    volumes:
      - mongodb-data:/data/db

volumes:
  mongodb-data:
```

`27017` é a porta padrão. O endereço `127.0.0.1` limita o acesso à própria máquina, e o volume preserva os dados quando o contêiner é recriado. A tag `<versao-fixada>` deve ser substituída por uma versão aprovada pela equipe; evitar `latest` torna os ambientes mais previsíveis.

Esse exemplo é para desenvolvimento. Em produção, não se deve expor o banco publicamente: use rede privada, autenticação, permissões mínimas, backups testados e monitoramento.

## Segurança e boas práticas

- habilite autenticação e crie usuários com apenas as permissões necessárias;
- mantenha o banco em rede privada e permita acesso somente aos serviços que precisam dele;
- use [[TLS]] para proteger conexões quando elas atravessarem redes não confiáveis;
- não coloque senhas no código, no [[Git]] ou em arquivos públicos;
- valide os dados no [[Backend]] e limite tamanho, tipos e campos recebidos;
- não monte filtros diretamente a partir de entradas sem validação;
- evite registrar documentos completos ou dados pessoais nos logs;
- crie índices para consultas importantes e analise consultas lentas;
- faça backups e teste a restauração, não apenas a criação dos arquivos;
- monitore memória, armazenamento, conexões, erros e crescimento das coleções.

Essas práticas fazem parte dos cuidados gerais de [[Segurança]].

## MongoDB na AWS

A AWS oferece o **Amazon DocumentDB**, um banco de documentos gerenciado com compatibilidade com uma parte das APIs e operações do MongoDB. Ele não é o MongoDB e não deve ser tratado como uma cópia totalmente compatível.

Antes de migrar, verifique as [diferenças de compatibilidade do Amazon DocumentDB](https://docs.aws.amazon.com/documentdb/latest/devguide/compatibility.html) e as [APIs e operações suportadas](https://docs.aws.amazon.com/documentdb/latest/devguide/mongo-apis.html). Recursos, comandos ou comportamentos específicos do MongoDB podem exigir adaptação.

Comparação simples:

- em uma [[VPS]], a equipe instala e administra o MongoDB: atualizações, segurança, armazenamento, backups, monitoramento e recuperação;
- no Amazon DocumentDB, a AWS administra boa parte da infraestrutura, o que reduz o trabalho operacional;
- o DocumentDB pode facilitar alta disponibilidade e backups gerenciados, mas tem custo, regras próprias da AWS e diferenças de compatibilidade;
- se for necessário o MongoDB exatamente como produto, também é possível operá-lo em infraestrutura própria ou contratar um serviço gerenciado específico, avaliando custo e responsabilidade.

## Resumo

MongoDB é um banco [[NoSQL]] de documentos BSON. Ele é útil quando os dados são naturalmente agrupados, o formato possui alguma flexibilidade e o sistema é modelado de acordo com suas consultas. Flexibilidade não substitui validação, índices, segurança, backups e uma escolha consciente entre MongoDB, [[PostgreSQL]] e outras soluções.
