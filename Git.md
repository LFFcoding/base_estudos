# Git

**Git** é um sistema de controle de versões (*version control system*). Ele registra as mudanças feitas nos arquivos de um projeto e permite voltar a versões anteriores quando necessário.

Uma analogia é imaginar um caderno com pontos de salvamento. A cada mudança importante, você cria uma fotografia do projeto. Se algo der errado, pode consultar uma fotografia antiga e entender o que mudou.

O Git funciona principalmente no computador da pessoa desenvolvedora. Ele pode trabalhar sozinho ou se conectar a serviços como o [[GitHub]].

## O que o Git resolve?

O Git ajuda a:

- saber quem alterou cada parte do código;
- entender quando e por que uma mudança foi feita;
- comparar versões de um arquivo;
- desfazer mudanças com segurança;
- trabalhar em funcionalidades diferentes ao mesmo tempo;
- juntar o trabalho de várias pessoas;
- manter um histórico do projeto.

Ele pode versionar o código do [[Backend]], do [[Frontend]], arquivos de configuração e documentação.

## Conceitos principais

- **Repositório (*repository*):** pasta que o Git acompanha e onde o histórico do projeto fica registrado.
- **Alteração (*change*):** modificação feita em um arquivo.
- **Commit:** registro de um conjunto de alterações. É como um ponto de salvamento com uma mensagem explicando o que foi feito.
- **Branch:** linha de desenvolvimento separada. Permite trabalhar em uma funcionalidade sem alterar imediatamente a linha principal.
- **Merge:** união das alterações de uma branch com outra.
- **Remote:** repositório externo, geralmente em um serviço como o [[GitHub]].

## Fluxo básico

Um fluxo comum é:

1. alterar os arquivos;
2. conferir o que mudou;
3. escolher quais arquivos farão parte do commit;
4. revisar as alterações escolhidas;
5. criar o commit;
6. enviar o trabalho para um remote, quando existir.

Exemplo:

```bash
git status
git diff
git add Backend.md
git diff --cached
git commit -m "explicado o conceito de backend"
```

Esses comandos fazem o seguinte:

- `git status` mostra arquivos modificados e não acompanhados;
- `git diff` mostra as alterações ainda não preparadas;
- `git add Backend.md` escolhe o arquivo que entrará no próximo commit;
- `git diff --cached` revisa o que foi escolhido;
- `git commit` registra a alteração com uma mensagem clara.

É melhor escolher os arquivos explicitamente do que usar `git add .` sem conferir o resultado. Assim, arquivos temporários ou alterações não relacionadas não entram por engano.

## Branches

Uma branch permite criar um caminho separado para uma mudança:

```bash
git switch -c assunto-requisicao

# faça as alterações e crie o commit
git add Requisição.md
git commit -m "refinada explicação de requisição"
```

Depois, esse trabalho pode ser revisado e unido à branch principal. O nome da branch deve explicar o objetivo da mudança.

## Git não é GitHub

Git é a ferramenta que controla o histórico do projeto. [[GitHub]] é um serviço online que pode guardar um repositório Git e facilitar a colaboração.

É possível usar Git sem GitHub. Também existem outros serviços que trabalham com Git, como GitLab e Bitbucket.

## Boas práticas

- criar commits pequenos e relacionados a uma única ideia, facilitando a revisão;
- escrever mensagens no imperativo e com objetivo claro, como `adiciona validação de usuário`;
- revisar `git diff` antes de preparar as alterações e `git diff --cached` antes do commit;
- criar branches para mudanças isoladas, evitando misturar funcionalidades;
- não versionar senhas, tokens, [[SSH#O que é uma chave SSH|chaves privadas]] ou arquivos gerados;
- usar um `.gitignore` para impedir que arquivos temporários e segredos sejam adicionados;
- evitar reescrever o histórico compartilhado sem combinar com a equipe;
- fazer commits frequentes o suficiente para formar pontos de recuperação úteis.

## Segurança

O Git registra o histórico. Se uma senha for commitada, apagar a senha do arquivo atual não remove automaticamente o valor dos commits antigos.

Por isso, nunca coloque segredos no código. Se um segredo for exposto, revogue ou troque-o imediatamente e, quando necessário, remova o valor do histórico com ferramentas adequadas.

## Resumo

> **Git é uma ferramenta que registra e organiza a história das mudanças de um projeto.**

Ele funciona como um histórico de pontos de salvamento, permitindo trabalhar com mais segurança e colaboração.
