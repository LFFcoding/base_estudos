# Mega-brain

Este repositório é um espaço de estudo sobre desenvolvimento de sistemas. Ele deve ser usado como um **vault** do Obsidian: uma pasta que guarda notas em Markdown e conecta os assuntos como um grande mapa mental.

A ideia é estudar cada conceito separadamente e, ao mesmo tempo, entender como os conceitos se relacionam. Por exemplo, o [[Frontend]] conversa com o [[Backend]] por meio de uma [[Requisição|requisição]], dentro da [[Stack]] da aplicação.

## Objetivo do repositório

O objetivo é formar uma base de conhecimento para atuar como **desenvolvedor** (*software developer*) e profissional de **DevOps** (*development and operations*).

Isso significa aprender a construir sistemas e também entender como colocá-los para funcionar, entregar versões com segurança, observar seu comportamento e resolver problemas em produção. A ideia é conectar programação, arquitetura, banco de dados, infraestrutura, automação e boas práticas em um único mapa de estudos.

O aprendizado terá como referência a stack definida no projeto: Java 17, Quarkus, Maven, PostgreSQL, Next.js, React, Docker Compose, Caddy e hospedagem em VPS. Quando um assunto de infraestrutura aparecer, também será apresentada a solução equivalente na AWS.

## Como abrir no Obsidian

1. Instale o [Obsidian](https://obsidian.md/), caso ainda não o tenha.
2. Abra o Obsidian.
3. Escolha **Open folder as vault** (*Abrir pasta como vault*).
4. Selecione a pasta deste repositório.
5. Abra qualquer arquivo `.md` para começar a estudar.

O Obsidian trabalha diretamente com os arquivos Markdown. Isso significa que as notas continuam sendo arquivos de texto simples e podem ser lidas fora do Obsidian ou versionadas com Git.

## Por onde começar

Comece pelo arquivo [[Stack]]. Ele apresenta a visão geral das tecnologias e mostra como as partes de uma aplicação se conectam.

Depois, siga esta ordem sugerida:

1. [[Frontend]]: entenda a parte que o usuário vê e utiliza.
2. [[Backend]]: entenda a parte que processa regras e dados.
3. [[Requisição]]: veja como frontend e backend conversam.
4. [[Framework]]: entenda a estrutura que ajuda a organizar uma aplicação.
5. [[Biblioteca]]: entenda os códigos reutilizáveis usados pelo projeto.

Essa ordem funciona como aprender a montar uma casa: primeiro você observa o projeto inteiro, depois conhece os cômodos, entende como eles conversam e, por fim, estuda as ferramentas usadas na construção.

## Como navegar pelos assuntos

Os assuntos relacionados usam **Wikilinks** (*links no formato do Obsidian*):

```markdown
[[Backend]]
```

No texto, esse link aparece como um link para a nota `Backend.md`. Para mostrar um texto diferente do nome do arquivo, use um **alias** (*apelido*):

```markdown
[[Requisição|requisições]]
```

Ao clicar no link, o Obsidian abre a nota relacionada. Se a nota ainda não existir, o Obsidian pode mostrar o link como uma nota não criada. Nesse caso, a nota pode ser criada depois, quando o assunto for estudado.

Também é possível visualizar as conexões no **Graph View** (*visualização gráfica*). Abra o gráfico pelo menu do Obsidian para ver os assuntos como pontos ligados por linhas.

## Como criar uma nova nota

Todo novo arquivo de estudo deve ser criado diretamente na pasta principal deste repositório. Não crie subpastas para separar grupos de assuntos, porque os caminhos das pastas podem deixar os links estranhos na leitura do Obsidian.

Use um nome claro e específico, como:

```text
Autenticacao.md
Docker Compose.md
Banco de dados.md
```

Uma nota pode seguir esta estrutura simples:

```markdown
# Nome do assunto

Explicação simples do conceito.

## Termos importantes

- Termo em português (*English term*): explicação curta.

## Como se relaciona com outros assuntos

Este assunto se conecta com [[Backend]] e [[Stack]].

## Exemplo de uso

Exemplo pequeno baseado em boas práticas.

## Boas práticas

- Prática recomendada e explicação do motivo.

## Resumo

Uma frase curta para lembrar a ideia principal.
```

## Regras para manter o mapa mental organizado

- Use palavras simples e explique como se estivesse ensinando um adolescente.
- Inclua os termos técnicos em inglês junto da explicação em português.
- Quando mencionar um assunto que já possui uma nota, transforme a menção em um Wikilink, como `[[Frontend]]`.
- Inclua links para assuntos relacionados ao longo do texto, no ponto em que eles forem citados.
- Apresente boas práticas e explique por que elas são importantes.
- Quando o assunto envolver comandos, código, objetos ou configurações, inclua pequenos exemplos de uso baseados nessas boas práticas.
- Para assuntos de infraestrutura, servidores ou hospedagem, explique também o equivalente na AWS e as diferenças principais.
- Evite criar links desnecessários. Cada link deve ajudar o leitor a continuar o estudo.

Atualmente, alguns pontos de partida são:

- [[Biblioteca]] e [[Framework]]: conceitos de ferramentas usadas no desenvolvimento;
- [[Frontend]] e [[Backend]]: partes principais de uma aplicação;
- [[Requisição]]: comunicação entre sistemas;
- [[Stack]]: conjunto de tecnologias usadas no projeto.

## Como usar com Git

O Git é um sistema de **version control** (*controle de versões*). Ele funciona como um histórico das mudanças, permitindo voltar e entender como o material evoluiu.

Depois de clonar o repositório, entre na pasta pelo [[Terminal]]:

```bash
cd caminho/para/Mega-brain
```

Depois de criar ou atualizar uma nota, confira as mudanças:

```bash
git status
git diff
```

Se tudo estiver correto, registre uma versão (**commit**) com uma mensagem clara:

```bash
git add README.md
git commit -m "docs: adiciona guia de uso do Obsidian"
git push
```

Boas práticas para o histórico:

- fazer commits pequenos, com uma mudança relacionada por vez;
- usar mensagens que expliquem o que foi alterado;
- revisar `git diff` antes do commit;
- não enviar senhas, chaves de API ou outros dados secretos;
- evitar enviar arquivos temporários ou configurações pessoais do computador.

## Objetivo final

Cada nota deve ajudar a entender um conceito e apontar para os próximos assuntos. Assim, o repositório deixa de ser apenas uma coleção de textos e se transforma em uma rede de conhecimentos conectados.
