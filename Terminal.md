# Terminal

Um **terminal** (*terminal emulator*) é um programa que permite conversar com o computador por meio de texto.

Ele normalmente abre uma janela onde podemos digitar comandos. O terminal envia esses comandos para um **shell**, que interpreta as instruções e pede ao sistema operacional para executá-las.

Uma analogia é uma conversa:

- o terminal é o telefone ou a janela da conversa;
- o shell é a pessoa que entende o que foi dito;
- o comando é o pedido;
- o sistema operacional é quem realiza o trabalho.

Por isso, terminal e shell não são exatamente a mesma coisa. O terminal é a interface; o shell é o programa que interpreta os comandos.

## Para que serve um terminal?

Um terminal pode ser usado para:

- navegar entre pastas;
- criar, copiar e remover arquivos;
- executar programas;
- instalar dependências;
- iniciar o [[Backend]];
- executar comandos do [[Git]], [[Maven]] e [[Docker]];
- acessar uma [[VPS]] por [[SSH]];
- automatizar tarefas com scripts (*scripts*).

Exemplos:

```bash
java -version
mvn test
git status
docker ps
```

Os comandos disponíveis dependem do sistema operacional e do shell usado dentro do terminal.

## Terminal, shell e comando

- **Terminal:** janela ou aplicativo que recebe e mostra texto;
- **Shell:** programa que interpreta comandos, como Bash, Zsh, Prompt de Comando ou PowerShell;
- **Comando:** instrução específica, como `git status` ou `docker ps`;
- **Script:** arquivo com vários comandos executados em sequência.

No macOS, é comum usar Terminal ou iTerm2 com Zsh. No Linux, são comuns os terminais com Bash ou Zsh. No Windows, é possível usar Prompt de Comando, PowerShell ou Windows Terminal.

## Boas práticas

- conferir a pasta atual antes de criar, alterar ou remover arquivos;
- usar `--help` ou a documentação antes de executar um comando desconhecido;
- ler o comando inteiro antes de pressionar Enter;
- ter cuidado com comandos que removem arquivos ou alteram muitas coisas;
- não colar comandos da internet sem entender o que fazem;
- não colocar senhas e tokens diretamente nos comandos;
- usar nomes de arquivos entre aspas quando eles tiverem espaços;
- manter ferramentas como Java, Maven, Git e Docker atualizadas;
- usar scripts versionados para tarefas repetitivas, em vez de depender apenas da memória;
- verificar mensagens de erro e o código de saída do comando.

## Resumo

> **Terminal é o programa que oferece uma janela de texto para interagir com o computador e executar comandos.**

Ele é a porta de entrada; o shell é quem entende os comandos digitados.
