# SSH

**SSH** significa *Secure Shell*, ou **shell seguro**. É um protocolo que permite acessar outro computador pela rede usando uma conexão criptografada.

Ele é muito usado para administrar servidores Linux, como uma [[VPS]]. Com SSH, é possível executar comandos, instalar programas, verificar logs e controlar aplicações sem estar fisicamente diante do servidor.

Uma analogia é um túnel seguro entre dois lugares:

- seu computador é o ponto de partida;
- o servidor é o destino;
- o túnel é a conexão SSH;
- a criptografia impede que outras pessoas entendam o que passa pelo túnel.

## Como uma conexão SSH funciona?

Quando você se conecta:

1. o cliente SSH do seu computador encontra o servidor;
2. o servidor apresenta sua identidade por meio de uma chave do servidor;
3. seu computador verifica essa identidade, especialmente na primeira conexão;
4. você prova que tem autorização usando uma senha ou uma [[SSH#O que é uma chave SSH|chave SSH]];
5. os dois lados criam uma sessão criptografada;
6. os comandos e as respostas passam pelo túnel seguro.

O SSH protege o caminho da comunicação, mas não transforma automaticamente um usuário mal configurado em um usuário seguro. A configuração do servidor continua sendo importante.

## Acesso com senha

Uma conexão pode pedir a senha do usuário:

```bash
ssh usuario@203.0.113.10
```

O endereço `203.0.113.10` é apenas um exemplo reservado para documentação. O comando tenta entrar no servidor usando o usuário `usuario`.

Senhas podem ser usadas, mas são mais vulneráveis a tentativas de descoberta e ataques automatizados. Para servidores, o acesso por chave SSH costuma ser preferível.

## O que é uma chave SSH?

Uma **chave SSH** é um par de arquivos usado para provar uma identidade sem enviar a senha pela rede. Esse par tem:

- **chave privada (*private key*):** fica somente com a pessoa ou sistema autorizado. Ela funciona como uma chave de casa e nunca deve ser compartilhada;
- **chave pública (*public key*):** pode ser cadastrada no servidor. Ela funciona como a fechadura que aceita a chave privada correspondente.

O servidor guarda a chave pública em um arquivo de usuários autorizados. Quando você tenta entrar, o SSH verifica se você possui a chave privada correspondente, sem precisar enviar o arquivo privado ao servidor.

Ter a chave pública não permite entrar sozinho. O segredo é a chave privada.

## Criando um par de chaves

Uma opção moderna é usar o algoritmo Ed25519:

```bash
ssh-keygen -t ed25519 -C "seu-nome@exemplo.com"
```

O comando cria, normalmente:

- `~/.ssh/id_ed25519`: chave privada;
- `~/.ssh/id_ed25519.pub`: chave pública.

Durante o processo, defina uma senha para a chave, chamada *passphrase*. Essa senha protege a chave privada caso o arquivo seja copiado.

Não envie `id_ed25519` para ninguém. O arquivo que pode ser cadastrado no servidor é somente `id_ed25519.pub`.

## Cadastrando a chave no servidor

Se o servidor permitir uma primeira entrada por senha, o comando abaixo pode copiar a chave pública para o usuário remoto:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@203.0.113.10
```

Depois, a conexão pode ser feita usando a chave privada:

```bash
ssh -i ~/.ssh/id_ed25519 usuario@203.0.113.10
```

O parâmetro `-i` informa qual chave privada deve ser usada. Em muitos sistemas, o SSH encontra automaticamente a chave em `~/.ssh`, mas indicar o arquivo deixa o exemplo mais claro.

## Configurando um nome para o servidor

Para não repetir todos os parâmetros, é possível criar uma configuração local em `~/.ssh/config`:

```sshconfig
Host minha-vps
    HostName 203.0.113.10
    User usuario
    IdentityFile ~/.ssh/id_ed25519
```

Depois disso, basta usar:

```bash
ssh minha-vps
```

Essa configuração fica no computador local. Ela não deve conter chaves privadas ou senhas escritas no arquivo.

## Boas práticas e segurança

- proteger a chave privada com uma *passphrase*, reduzindo o impacto de uma cópia indevida;
- nunca enviar a chave privada por mensagem, e-mail, repositório ou chamado de suporte;
- usar uma chave diferente para cada pessoa ou finalidade, facilitando a revogação de acessos;
- remover a chave pública do servidor quando uma pessoa deixar de precisar do acesso;
- conferir a impressão digital (*fingerprint*) do servidor na primeira conexão, evitando aceitar um servidor falso;
- usar um usuário comum e `sudo` quando necessário, em vez de trabalhar sempre como `root`;
- desabilitar o login direto de `root` e o acesso por senha somente depois de testar a chave em outra sessão;
- limitar a porta SSH no firewall para redes ou endereços confiáveis quando possível;
- manter o sistema operacional e o servidor SSH atualizados;
- guardar cópias de segurança das chaves privadas em local criptografado, sem deixar cópias espalhadas;
- usar um agente SSH (*ssh-agent*) para não digitar a *passphrase* a cada comando, mantendo o agente protegido.

As permissões dos arquivos também importam:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Esses comandos deixam a pasta acessível somente ao usuário, protegem a chave privada e permitem que a chave pública seja lida.

## SSH na AWS

Em uma instância Linux do **Amazon EC2** (*Elastic Compute Cloud*), é comum usar um **key pair** da AWS, que é o equivalente ao par de chaves SSH. A chave pública é instalada na instância e a chave privada, geralmente baixada como arquivo `.pem`, fica com a pessoa responsável.

Um acesso pode ter este formato:

```bash
ssh -i minha-chave.pem usuario@203.0.113.10
```

O nome do usuário depende da imagem Linux escolhida. Por segurança, a chave privada deve ficar protegida e a regra de rede deve permitir SSH somente de origens necessárias.

Outra opção da AWS é o **AWS Systems Manager Session Manager**. Ele permite abrir uma sessão no EC2 usando permissões do IAM e um agente instalado na instância, sem precisar expor a porta SSH para a internet. Essa opção reduz a superfície de ataque, mas exige configuração de IAM, agente e conectividade com os serviços da AWS.

## SSH e GitHub

O SSH também pode autenticar o acesso ao [[GitHub]]. Nesse caso, uma chave pública é cadastrada na conta e a chave privada fica no computador local.

Isso permite usar URLs SSH para operações do [[Git]], como baixar ou enviar alterações, sem colocar uma senha diretamente no comando.

Mesmo no GitHub, a chave privada continua sendo secreta. Se ela for exposta, deve ser removida da conta e substituída.

## Resumo

> **SSH é um protocolo para acessar computadores remotamente com uma conexão criptografada.**

> **Uma chave SSH é um par de chaves pública e privada usado para autenticar esse acesso com mais segurança do que uma senha.**
