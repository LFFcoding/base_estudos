# GitHub

**GitHub** é uma plataforma online para armazenar repositórios Git e colaborar no desenvolvimento de projetos.

Uma analogia simples:

- [[Git]] é o caderno de histórico que fica no computador;
- GitHub é uma biblioteca compartilhada na internet onde esse caderno pode ser guardado;
- uma pessoa envia suas mudanças para a biblioteca e outras pessoas podem revisar, comentar e contribuir.

O GitHub não substitui o Git. Ele usa o Git para controlar as versões, mas oferece recursos adicionais para o trabalho em equipe.

## O que o GitHub oferece?

- **Repositórios (*repositories*):** espaços para armazenar código, documentação e histórico;
- **Branches:** caminhos separados para desenvolver funcionalidades;
- **Pull requests:** propostas para revisar e juntar alterações;
- **Code review:** análise do código por outras pessoas antes da união;
- **Issues:** registros de tarefas, dúvidas, erros e melhorias;
- **GitHub Actions:** automações, como executar testes e publicar aplicações;
- **Permissões:** controle sobre quem pode visualizar ou alterar um projeto;
- **README:** arquivo que apresenta o objetivo e a forma de usar o projeto.

Um repositório pode guardar o código do [[Backend]], do [[Frontend]] e arquivos da [[Stack]] do projeto.

## Git local e GitHub remoto

O repositório local fica no computador. O repositório remoto (*remote repository*) fica no GitHub.

O fluxo costuma ser:

1. baixar o projeto do GitHub;
2. criar uma branch local;
3. fazer alterações e commits com [[Git]];
4. enviar a branch para o GitHub;
5. abrir um pull request;
6. revisar e integrar as alterações.

Exemplo:

```bash
git clone https://github.com/exemplo/projeto.git
cd projeto
git switch -c melhora-documentacao

# faça as alterações
git add Requisição.md
git commit -m "melhora documentação de requisição"
git push -u origin melhora-documentacao
```

Nesse exemplo:

- `git clone` baixa uma cópia do repositório;
- `cd` entra na pasta do projeto;
- `git switch -c` cria e acessa uma nova branch;
- `git add` e `git commit` registram a alteração localmente;
- `git push` envia a branch para o GitHub.

Depois do `push`, a pessoa pode abrir um pull request no GitHub para pedir a revisão e a integração da branch.

## Pull request

Um **pull request** (*proposta de integração*) é uma solicitação para juntar as alterações de uma branch em outra, normalmente na branch principal.

Ele permite:

- explicar o que foi alterado;
- mostrar os arquivos modificados;
- discutir decisões com a equipe;
- executar verificações automáticas;
- receber aprovação antes da integração.

Um pull request não é apenas um botão para juntar código. Ele é um espaço de comunicação e revisão que ajuda a encontrar problemas antes que cheguem à versão principal.

## Boas práticas no GitHub

- proteger a branch principal, exigindo revisão antes de integrar alterações;
- abrir pull requests pequenos e com objetivo claro, facilitando a análise;
- explicar no pull request o problema, a solução e como testar;
- executar testes e verificações automáticas antes da integração;
- revisar permissões e dar a cada pessoa somente o acesso necessário;
- ativar autenticação em dois fatores (*two-factor authentication* ou 2FA);
- usar [[SSH#O que é uma chave SSH|chaves SSH]] ou tokens de acesso com escopo limitado, em vez de colocar senhas nos comandos;
- nunca publicar tokens, senhas, chaves privadas ou dados pessoais no repositório;
- usar um arquivo README claro para ajudar outras pessoas a entenderem o projeto;
- remover ou corrigir rapidamente qualquer segredo exposto, lembrando que ele pode permanecer no histórico.

## GitHub Actions e [[CI/CD]]

O GitHub Actions é o recurso de automação (*automation*) do GitHub. Ele pode executar tarefas de [[CI/CD]] quando algo acontece no repositório, como:

- rodar testes quando um pull request é aberto;
- verificar estilo e qualidade do código;
- construir uma aplicação;
- publicar uma nova versão.

Uma automação deve falhar quando encontrar um problema importante. Isso impede que código quebrado seja integrado ou publicado sem que a equipe perceba.

## GitHub não é hospedagem da aplicação

Guardar o código no GitHub não significa que a aplicação esteja disponível para os usuários. O GitHub hospeda principalmente o repositório e as ferramentas de colaboração.

A aplicação ainda precisa ser executada e publicada em um ambiente próprio, como uma [[VPS]], usando os serviços definidos na [[Stack]].

## Resumo

> **GitHub é uma plataforma online que usa Git para armazenar código e facilitar a colaboração.**

O Git controla o histórico; o GitHub oferece um espaço compartilhado para revisar, discutir, automatizar e integrar as mudanças.
