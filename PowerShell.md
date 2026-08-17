# PowerShell

**PowerShell** é um shell e uma linguagem de script (*scripting language*) criada para automatizar tarefas e administrar sistemas.

Ele surgiu no Windows, mas também pode ser usado no macOS e no Linux. Assim como o [[Prompt de comando]], ele normalmente funciona dentro de um [[Terminal]].

Uma diferença importante é que o PowerShell trabalha com **objetos**, não apenas com texto. Isso permite filtrar e organizar informações com mais precisão.

## Comandos e cmdlets

Os comandos do PowerShell são chamados de **cmdlets** (*command-lets*). Eles normalmente seguem o formato `Verbo-Substantivo`, que facilita entender sua função.

```powershell
Get-Location
Get-ChildItem
Set-Location .\exemplos
Get-Process
Get-Help Get-Process
```

Esses comandos fazem o seguinte:

- `Get-Location` mostra a pasta atual;
- `Get-ChildItem` lista arquivos e pastas;
- `Set-Location` muda de pasta;
- `Get-Process` lista processos em execução;
- `Get-Help` mostra ajuda sobre um comando.

## Pipeline de objetos

O caractere `|` envia o resultado de um comando para o próximo comando. No PowerShell, esse resultado é tratado como objeto:

```powershell
Get-Process | Where-Object CPU -gt 100
```

Esse exemplo lista processos cujo uso acumulado de CPU é maior que 100. O segundo comando consegue acessar a propriedade `CPU` diretamente.

Também é possível usar o PowerShell para executar ferramentas do projeto:

```powershell
java -version
mvn test
git status
docker ps
```

Esses comandos chamam Java, Maven, Git e Docker instalados no computador.

## Variáveis e scripts

Variáveis no PowerShell começam com `$`:

```powershell
$ambiente = "desenvolvimento"
Write-Output "Ambiente: $ambiente"
```

Scripts normalmente usam a extensão `.ps1`. Eles podem automatizar tarefas como preparar um ambiente, executar testes ou iniciar serviços.

## PowerShell e Prompt de Comando

O [[Prompt de comando]] usa comandos tradicionais do `cmd.exe`, como `dir` e `set`. O PowerShell usa cmdlets como `Get-ChildItem` e `Set-Location`, além de conseguir executar muitos comandos antigos do Windows.

O PowerShell é mais adequado para automação complexa porque possui objetos, funções, módulos e recursos para tratar erros e dados estruturados.

## Boas práticas e segurança

- usar `Get-Help` e `Get-Command` para entender os comandos disponíveis;
- revisar scripts `.ps1` antes de executá-los;
- não usar `-ExecutionPolicy Bypass` sem entender o risco e a política do ambiente;
- não executar scripts baixados da internet sem verificar sua origem;
- não guardar senhas, tokens ou chaves privadas em arquivos de script;
- tratar erros e verificar se os comandos terminaram corretamente;
- usar parâmetros em scripts, evitando editar o código para cada ambiente;
- testar scripts em um ambiente seguro antes de executá-los em servidores;
- executar o PowerShell como administrador somente quando a tarefa realmente exigir;
- manter o PowerShell e o sistema operacional atualizados.

## Resumo

> **PowerShell é um shell moderno e uma linguagem de scripts que permite executar comandos e automatizar tarefas usando objetos.**

Ele é especialmente útil para administração de sistemas, automação e tarefas repetitivas.
