# Diretrizes do projeto

## Objetivo

Este repositório é um espaço de estudo sobre conceitos de desenvolvimento de sistemas.

## Stack alvo de estudos

A stack principal usada nos estudos será:

- **Backend:** Java 17 com Quarkus e Maven (`mvn`);
- **Banco de dados:** PostgreSQL;
- **Frontend:** Next.js com React;
- **Execução local e implantação:** Docker Compose;
- **Servidor web e proxy:** Caddy;
- **Hospedagem:** VPS.

## Conexão entre os assuntos

O objetivo deste repositório é formar um grande mapa mental, conectando os conhecimentos necessários para formar um bom profissional.

Sempre que um novo arquivo sobre um assunto for criado:

- incluir links para outros arquivos que tenham assuntos relevantes relacionados;
- utilizar a sintaxe de links do Obsidian, como `[[Nome do arquivo]]`;
- transformar em link cada assunto já existente que for citado, usando o link ao longo do texto, no ponto em que o assunto aparecer;
- manter as conexões claras e úteis para facilitar a navegação entre os conhecimentos.

## Organização dos arquivos

Os arquivos de estudo devem permanecer na pasta principal do repositório, sem serem separados em subpastas por grupo de assunto.

Essa escolha existe porque os links com a sintaxe do Obsidian podem exibir o caminho da pasta ou deixar as palavras estranhas no texto quando os arquivos são organizados em subpastas. Mantendo os arquivos na pasta principal, os links ficam simples e naturais, como `[[Frontend]]`, preservando a leitura e a navegação do mapa mental.

Ao criar um novo arquivo:

- criar o arquivo diretamente na pasta principal;
- usar os links do Obsidian para conectar assuntos relacionados, em vez de criar subpastas;
- manter os nomes dos links simples, sem incluir caminhos de pastas, para que apareçam naturalmente nos textos.

## Commits no Git

Quando o usuário solicitar explicitamente, o agente deve revisar e criar um **commit** (*registro de uma versão*) com as alterações relacionadas aos assuntos criados ou atualizados naquele pedido.

Antes do commit, o agente deve:

- verificar o estado do repositório com `git status`;
- revisar as alterações com `git diff`;
- incluir somente as mudanças relacionadas ao pedido atual;
- usar uma mensagem de commit no formato:

```text
criado(s): assunto1, assunto2; atualizado(s): assunto3, assunto4
```

Os nomes devem identificar os assuntos estudados, e não apenas repetir os nomes dos arquivos. Quando uma das categorias não tiver itens, usar `nenhum`. Por exemplo:

```text
criado(s): Docker Compose; atualizado(s): Stack, Backend
```

## Regra para os arquivos

Todo arquivo criado ou atualizado para explicar um assunto deve:

- usar palavras simples e fáceis de entender;
- explicar os conceitos de forma que até um adolescente consiga acompanhá-los;
- utilizar boas analogias sempre que elas ajudarem a tornar o assunto mais claro;
- mencionar os termos técnicos usados em inglês, preferencialmente junto da explicação em português;
- indicar as boas práticas relacionadas ao assunto, explicando de forma simples por que elas são importantes;
- quando o assunto envolver comandos, códigos, objetos, configurações ou outros elementos práticos, incluir pequenos exemplos de uso baseados nas boas práticas apresentadas e explicar de forma simples o que cada exemplo faz;
- quando o assunto estiver relacionado a infraestrutura, servidores ou hospedagem, como uma VPS, apresentar também o serviço ou a solução equivalente na AWS e explicar as principais diferenças quando não houver uma correspondência exata.
