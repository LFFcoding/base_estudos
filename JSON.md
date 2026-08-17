# JSON

**JSON** significa *JavaScript Object Notation*. É um formato de texto usado para representar e trocar dados entre programas.

Apesar do nome, JSON não é uma linguagem de programação. Ele é como um formulário padronizado: um sistema preenche os campos, envia o texto e outro sistema consegue entender os mesmos dados.

## Onde o JSON aparece?

JSON é muito usado:

- no body de uma [[Requisição]] HTTP entre o [[Frontend]] e o [[Backend]];
- nas respostas de APIs;
- em arquivos de configuração;
- para transportar dados entre serviços;
- dentro do payload de um [[JWT]], embora JSON sozinho não ofereça segurança.

Quando o conteúdo de uma requisição é JSON, o header normalmente informa:

```http
Content-Type: application/json
```

## Estrutura básica

JSON possui seis tipos de valor:

- **objeto (*object*)**: conjunto de pares `chave: valor`, entre `{}`;
- **array**: lista de valores, entre `[]`;
- **string**: texto entre aspas duplas;
- **number**: número inteiro ou decimal;
- **boolean**: `true` ou `false`;
- **null**: ausência intencional de valor.

Exemplo:

```json
{
  "id": 42,
  "nome": "Ana",
  "ativo": true,
  "email": null,
  "permissoes": ["pedidos:read", "pedidos:create"],
  "endereco": {
    "cidade": "São Paulo",
    "pais": "Brasil"
  }
}
```

Nesse exemplo, o JSON representa uma pessoa com número de identificação, nome, situação, permissões e um objeto de endereço.

## Regras da sintaxe

- chaves e textos usam aspas duplas, como `"nome"`;
- cada par de propriedade usa `:` entre a chave e o valor;
- itens consecutivos são separados por vírgula;
- o último item não deve ter vírgula sobrando;
- comentários não fazem parte do JSON padrão;
- os nomes das chaves diferenciam maiúsculas de minúsculas;
- o arquivo deve ser válido para que o programa consiga interpretá-lo.

JSON válido:

```json
{
  "nome": "Ana",
  "idade": 16
}
```

JSON inválido:

```json
{
  nome: 'Ana',
  "idade": 16,
}
```

O segundo exemplo usa aspas simples, não coloca aspas em `nome` e deixa uma vírgula depois do último campo.

## JSON em uma API

Uma aplicação pode enviar este body ao [[Backend]]:

```json
{
  "produtoId": 10,
  "quantidade": 2
}
```

O backend deve interpretar o JSON, validar os campos, verificar a autorização e só depois executar a operação. O JSON transporta os dados; ele não substitui as regras de negócio nem a segurança.

Uma resposta também pode ser JSON:

```json
{
  "id": 301,
  "status": "criado"
}
```

## JSON e objetos do programa

Um programa pode transformar um objeto de sua linguagem em JSON. Essa operação é chamada de **serialização** (*serialization*). O caminho inverso é a **desserialização** (*deserialization*).

Por exemplo:

```text
objeto do programa -> serialização -> texto JSON
texto JSON         -> desserialização -> objeto do programa
```

O resultado depende das regras da linguagem e da biblioteca utilizada. Nem todo tipo de objeto pode ser convertido automaticamente sem perda de informação.

## Boas práticas

- defina nomes de campos consistentes, como `produtoId` ou `produto_id`, e mantenha o padrão;
- documente o formato esperado pela API;
- valide campos obrigatórios, tipos, tamanhos e valores permitidos no backend;
- retorne mensagens de erro claras sem revelar senhas, tokens ou detalhes internos;
- não coloque segredos no JSON, mesmo que o transporte use HTTPS;
- evite enviar dados desnecessários, reduzindo tamanho e exposição;
- mantenha compatibilidade ao alterar uma resposta consumida por outros sistemas;
- considere versionamento quando uma mudança quebrar clientes existentes;
- use UTF-8 e informe corretamente o `Content-Type`;
- trate erros de análise (*parse errors*) sem derrubar o serviço.

JSON é simples de ler e muito útil para integração, mas um formato simples ainda precisa de contrato, validação e cuidado com os dados transportados.
