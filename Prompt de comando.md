# Prompt de comando

O **Prompt de Comando** (*Command Prompt*), também conhecido como `cmd.exe`, é um shell tradicional do Windows. Ele interpreta comandos de texto e solicita tarefas ao sistema operacional.

Ele costuma ser aberto dentro de uma janela de [[Terminal]]. O terminal é a janela; o Prompt de Comando é o programa que entende os comandos digitados nela.

Uma analogia é um atendente que conhece um conjunto específico de palavras. Você faz um pedido, ele interpreta a frase e pede ao Windows para realizar a tarefa.

## Comandos comuns

```bat
cd C:\Projetos\estudo
dir
echo Olá
mkdir exemplos
java -version
mvn test
docker ps
```

Esses comandos fazem o seguinte:

- `cd` muda de pasta;
- `dir` lista os arquivos e pastas;
- `echo` mostra um texto;
- `mkdir` cria uma pasta;
- `java -version` mostra a versão do Java;
- `mvn test` executa os testes do projeto Maven;
- `docker ps` lista containers Docker em execução.

O Prompt de Comando usa uma sintaxe diferente de shells Unix. Por exemplo, `dir` é usado para listar arquivos, enquanto Bash e Zsh normalmente usam `ls`.

## Variáveis e scripts

Uma variável no Prompt de Comando usa o formato `%NOME%`:

```bat
set AMBIENTE=desenvolvimento
echo %AMBIENTE%
```

Vários comandos podem ser salvos em um arquivo `.bat` ou `.cmd` para formar um script executável pelo Windows.

## Prompt de Comando ou PowerShell?

O Prompt de Comando é mais antigo e simples. O [[PowerShell]] é mais moderno, possui comandos próprios e trabalha com objetos, o que facilita scripts mais completos.

Para tarefas simples e programas antigos do Windows, o Prompt de Comando pode ser suficiente. Para automação, administração e scripts mais elaborados, o PowerShell costuma oferecer mais recursos.

## Boas práticas

- abrir o terminal na pasta correta antes de executar comandos;
- usar `dir` para conferir os arquivos antes de alterá-los;
- evitar executar o Prompt de Comando como administrador sem necessidade;
- revisar arquivos `.bat` e `.cmd` antes de executá-los;
- não salvar senhas ou tokens em scripts;
- testar scripts em uma pasta de estudo antes de usá-los em projetos importantes;
- preferir PowerShell quando a tarefa exigir tratamento estruturado de dados;
- não ignorar mensagens de erro do Windows.

## Resumo

> **Prompt de Comando é o shell tradicional do Windows, usado para executar comandos de texto e scripts simples.**

Ele é uma opção de terminal, mas não é o único shell disponível no Windows.
