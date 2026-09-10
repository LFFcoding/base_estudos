# O que é Godot Engine?

**Godot Engine** é uma engine (motor) gratuita e de código aberto para criar jogos e experiências interativas em 2D e 3D. Ela oferece editor, renderização, física, áudio, animação, interface, entrada do jogador, ferramentas de exportação e recursos de multiplayer.

Uma engine funciona como uma oficina pronta. Em vez de construir do zero o sistema que desenha imagens, detecta colisões e reproduz sons, o desenvolvedor usa essas ferramentas para se concentrar nas regras e na experiência do jogo.

O Godot é multiplataforma e usa uma licença MIT permissiva. Um projeto pode ser desenvolvido com GDScript, C#, C++ por meio de extensões e outras ferramentas compatíveis com o ecossistema.

## Para que serve?

Godot pode ser usado para criar:

- jogos 2D, 3D e experiências interativas;
- protótipos rápidos;
- interfaces e ferramentas visuais;
- simulações e aplicações educacionais;
- experiências para computador, celular e web, conforme os recursos usados;
- clientes que conversam com um [[Backend]] por HTTP ou outros protocolos.

## Os quatro conceitos principais

A estrutura do Godot pode ser entendida por quatro ideias:

1. **Nodes:** blocos básicos do jogo;
2. **Scenes:** árvores de nodes salvas e reutilizáveis;
3. **Scene Tree:** a árvore de nodes que está em execução;
4. **Signals:** mensagens que avisam que um evento aconteceu.

Uma analogia: nodes são peças de montar, uma scene é um brinquedo montado com essas peças, a Scene Tree é o brinquedo completo em uso e signals são bilhetes enviados de uma peça para outra.

## Nodes

Um **Node** é a unidade básica do Godot. Ele possui nome, propriedades, pode ter filhos e pode receber chamadas do ciclo de vida da engine.

Alguns nodes comuns são:

- `Node`: base para comportamentos gerais;
- `Node2D`: base para objetos em um espaço 2D;
- `Node3D`: base para objetos em um espaço 3D;
- `Control`: base para elementos de interface;
- `Sprite2D`: exibe uma imagem 2D;
- `Camera2D` e `Camera3D`: definem o ponto de vista;
- `CharacterBody2D` e `CharacterBody3D`: ajudam a controlar personagens;
- `CollisionShape2D` e `CollisionShape3D`: definem formas de colisão;
- `Timer`: dispara eventos depois de um intervalo.

Os nodes podem ser organizados em uma hierarquia. Um node filho herda a organização e, em alguns casos, a transformação do node pai. Essa composição permite criar comportamentos complexos usando peças menores.

## Scenes

Uma **Scene** é uma árvore de nodes que pode ser salva e reutilizada. Uma scene pode representar:

- um personagem;
- uma arma;
- uma porta;
- um menu;
- um inimigo;
- uma fase completa.

Ao salvar uma scene, ela pode ser instanciada várias vezes em outras scenes. Assim, uma única cena de inimigo pode ser usada para criar dezenas de inimigos sem copiar toda a estrutura manualmente.

Todo projeto precisa de uma cena principal (*main scene*), que é carregada quando o jogo começa. Cenas menores podem ser carregadas ou removidas durante a execução.

## Scene Tree

A **Scene Tree** é a árvore de nodes existente naquele momento. Ela representa a composição atual do jogo:

```text
Mundo
├── Jogador
│   ├── Sprite2D
│   ├── CollisionShape2D
│   └── Camera2D
└── Inimigos
    ├── InimigoA
    └── InimigoB
```

É possível encontrar nodes pelo caminho, adicionar filhos e remover partes da árvore. Quando um node não é mais necessário, `queue_free()` agenda sua remoção com segurança ao final do processamento adequado.

## Scripts e GDScript

**GDScript** é a linguagem criada para trabalhar com o Godot. Sua sintaxe é simples e lembra Python, mas ela possui recursos próprios para nodes, signals e cenas.

Um script simples para mover um objeto pode ser:

```gdscript
extends Node2D

@export var velocidade: float = 200.0

func _process(delta: float) -> void:
    position.x += velocidade * delta
```

Nesse exemplo:

- `extends Node2D` indica que o script adiciona comportamento a um node 2D;
- `@export` permite ajustar a velocidade no Inspector;
- `_process` é chamado durante o processamento de cada quadro;
- `delta` representa o tempo desde o quadro anterior;
- multiplicar pela velocidade e por `delta` torna o movimento mais consistente em diferentes taxas de quadros.

Use anotações de tipo, como `velocidade: float` e `delta: float`, sempre que elas deixarem a intenção mais clara. A tipagem ajuda o editor a sugerir recursos e a identificar erros mais cedo.

## C# no Godot

Também é possível programar com [[C#]]. Nesse caso, o projeto usa a edição do Godot com suporte a .NET, e os scripts seguem as APIs e convenções do C#:

```csharp
using Godot;

public partial class Mover : Node2D
{
    [Export]
    public float Velocidade { get; set; } = 200f;

    public override void _Process(double delta)
    {
        Position += Vector2.Right * Velocidade * (float)delta;
    }
}
```

GDScript costuma oferecer o caminho mais direto para aprender a engine. C# pode ser uma boa escolha para quem já trabalha com .NET ou prefere sua estrutura de tipos e ferramentas. A escolha deve considerar a experiência da equipe, os plugins necessários e as plataformas de destino.

## C++ e extensões

O núcleo do Godot é escrito principalmente em C++. Um projeto pode usar [[C++]] por meio de módulos ou extensões, como GDExtension, quando precisa integrar uma biblioteca nativa, criar uma parte de alto desempenho ou acessar recursos específicos da plataforma.

Para a maioria dos comportamentos de gameplay, GDScript ou C# costuma ser suficiente. Usar C++ adiciona complexidade de compilação, distribuição e gerenciamento de memória; por isso, vale medir o problema antes de escolher essa opção.

## Signals

**Signals** são mensagens emitidas quando algo acontece. Um botão pode emitir um sinal quando é pressionado, e um personagem pode emitir um sinal quando perde toda a vida.

```gdscript
extends Node

signal vida_esgotada

var vida: int = 100

func receber_dano(valor: int) -> void:
    vida -= valor

    if vida <= 0:
        vida_esgotada.emit()
```

Outro node pode se conectar ao sinal e reagir sem precisar conhecer todos os detalhes do node que o emitiu. Isso reduz o acoplamento e é uma aplicação do [[Design Pattern]] Observer.

Boas práticas para signals:

- use nomes que expressem o evento, como `vida_esgotada`;
- conecte o sinal no local em que a relação fica mais clara;
- desconecte conexões manuais quando o ciclo de vida exigir;
- evite criar uma rede de sinais tão complexa que o fluxo fique impossível de acompanhar;
- use signals para comunicar eventos, não para esconder regras importantes.

## Resources

Um **Resource** é um objeto de dados reutilizável. Pode guardar configurações de um item, atributos de um personagem, dados de uma arma ou parâmetros de um nível.

Separar dados da lógica traz vantagens:

- vários nodes podem compartilhar a mesma configuração;
- designers podem ajustar valores no editor;
- o código fica menos dependente de números espalhados;
- itens e personagens podem ser criados a partir de dados diferentes.

Por exemplo, uma espada pode ter dano, alcance e preço em um Resource, enquanto o script da espada cuida apenas de aplicar essas regras.

Não coloque segredos ou credenciais em Resources exportados. Tudo que vai para o cliente pode ser lido ou alterado por alguém com acesso ao jogo.

## Física e colisões

O Godot fornece nodes para corpos físicos, áreas e formas de colisão. É importante distinguir:

- **collision:** impede ou participa do cálculo de movimento;
- **area/overlap:** detecta a entrada ou saída de objetos em uma região;
- **rigid body:** reage às forças da simulação;
- **character body:** representa um corpo controlado por regras de movimento do jogo.

Use formas de colisão simples e escolha o tipo de corpo de acordo com o comportamento desejado. Uma caixa ou cápsula costuma ser mais barata e previsível do que uma forma detalhada desnecessária.

## Ciclo de vida

Alguns métodos importantes são:

- `_enter_tree`: chamado quando o node entra na Scene Tree;
- `_ready`: chamado quando o node e seus filhos estão prontos;
- `_process`: usado para lógica executada a cada quadro;
- `_physics_process`: usado para lógica ligada ao passo fixo da física;
- `_exit_tree`: chamado quando o node sai da Scene Tree.

Use `_physics_process` para movimentação e lógica que precisa acompanhar a simulação física. Evite colocar trabalho pesado em `_process` sem medir o impacto.

## Interface e entrada do jogador

Nodes `Control` formam a interface do usuário. É possível criar menus, botões, barras de vida e inventários usando containers, labels, texturas e temas.

Configure ações no Input Map, como `mover_esquerda`, `pular` e `interagir`, em vez de espalhar teclas diretamente pelo código:

```gdscript
if Input.is_action_just_pressed("pular"):
    pular()
```

Dessa forma, a mesma ação pode ser associada a teclado, controle ou tela sensível ao toque sem reescrever toda a lógica.

## Mapa procedural

O Godot pode gerar mapas por algoritmos usando nodes, scenes, tiles, meshes e Resources. Um fluxo possível é:

1. escolher uma seed;
2. gerar uma grade ou ruído;
3. transformar os valores em terrenos e obstáculos;
4. instanciar scenes para objetos especiais;
5. validar caminhos e pontos de interesse;
6. carregar o mapa por partes quando ele for muito grande.

Esse assunto se conecta a [[Mapa procedural]]. Para muitos objetos iguais, procure estratégias de instancing, reutilização e carregamento sob demanda em vez de criar tudo de uma vez.

## Multiplayer

O Godot oferece uma API de alto nível para multiplayer. Em uma arquitetura cliente-servidor:

- o servidor mantém o estado autoritativo;
- clientes enviam intenções e recebem atualizações;
- RPCs (*Remote Procedure Calls*) transportam chamadas entre máquinas;
- propriedades e eventos podem ser sincronizados conforme a configuração da aplicação.

Nunca confie no cliente para decidir sozinho dano, dinheiro, inventário ou permissões. O servidor deve validar as ações, mesmo que o cliente também faça uma validação para melhorar a experiência visual.

Planeje a rede cedo. Adicionar multiplayer depois que todo o jogo foi construído como se houvesse somente uma máquina pode exigir mudanças profundas.

## Comunicação com um backend

O jogo pode fazer uma [[Requisição]] para uma API e receber dados em [[JSON]], como perfil, placar ou configuração. O cliente Godot é apenas uma das partes do sistema; regras que precisam de confiança devem ficar no [[Backend]].

Boas práticas:

- não coloque tokens permanentes ou segredos no cliente;
- valide respostas e trate erros de rede;
- considere timeout, perda de conexão e repetição de requisições;
- não confie em valores recebidos do servidor sem verificar o formato;
- mantenha operações sensíveis no servidor;
- registre e trate falhas sem expor dados pessoais.

## Exportação

Exportar é transformar o projeto em uma versão executável para uma plataforma. O Godot usa presets de exportação, que definem destino, arquitetura, recursos e configurações do build.

Antes de exportar:

- defina e teste a cena principal;
- use um preset para cada plataforma;
- confirme que as templates de exportação estão instaladas;
- verifique caminhos, permissões e arquivos incluídos;
- teste a versão exportada, não somente o editor;
- não inclua credenciais ou arquivos de desenvolvimento.

Também é possível automatizar exportações pelo terminal:

```bash
godot --path /caminho/do/projeto --editor
godot --path /caminho/do/projeto --export-release "Linux" jogo.x86_64
```

O nome do preset e o caminho de saída dependem do projeto. Builds automatizados podem fazer parte de um fluxo de [[CI-CD]].

## Godot e controle de versão

Use [[Git]] para versionar scripts, scenes, Resources, configurações e arquivos necessários para reproduzir o projeto.

Em geral, a pasta `.godot` contém dados gerados ou cache local e não deve ser tratada como conteúdo principal do projeto. Defina um `.gitignore` adequado e confirme com a equipe quais assets grandes precisam de Git LFS ou de outra solução.

Faça commits pequenos e claros. Isso facilita descobrir quando uma scene ou script começou a apresentar um problema.

## Testes e performance

Projetos Godot também precisam de [[Testes]]. Além de testar funções isoladas, vale verificar movimento, colisão, carregamento de scenes, salvamento, interface, exportação e multiplayer.

Para desempenho:

- use o profiler antes de otimizar;
- evite trabalho caro em `_process` e `_physics_process`;
- desative processamento de nodes que não estão ativos;
- reutilize objetos quando a criação e destruição forem frequentes;
- carregue assets por demanda em mundos grandes;
- mantenha cenas e scripts com responsabilidades claras;
- teste em hardware próximo ao do público.

Princípios de [[SOLID]] e [[Design Pattern]] podem ajudar a separar responsabilidades, mas o código deve continuar simples para a equipe que vai mantê-lo.

## Godot, Unity e Unreal

Godot, [[Unity Engine]] e [[Unreal Engine]] resolvem problemas parecidos, mas têm ecossistemas diferentes:

- Godot usa principalmente GDScript e também suporta C# e extensões C++;
- Unity usa C# como linguagem principal de scripts;
- Unreal combina C++ com Blueprints;
- Godot organiza o jogo em nodes e scenes;
- Unity trabalha com GameObjects, Components e Scenes;
- Unreal trabalha com Actors, Components e Levels;
- Godot é livre e de código aberto sob licença MIT, enquanto as outras engines têm modelos de licença e distribuição diferentes.

Nodes e Components cumprem papéis parecidos: ambos permitem montar objetos a partir de partes menores. Ainda assim, as APIs, ferramentas e formas de gerenciamento de memória não são iguais.

## Boas práticas resumidas

- aprenda a relação entre nodes, scenes, Scene Tree e signals;
- mantenha cenas pequenas e reutilizáveis;
- use Resources para separar dados de comportamento;
- dê nomes claros a nodes, scripts, ações e signals;
- prefira eventos e signals quando não for necessário verificar algo a cada quadro;
- use tipagem explícita quando ela melhorar a clareza;
- evite lógica global excessiva em Autoloads;
- mantenha regras importantes do multiplayer no servidor;
- não guarde segredos no projeto exportado;
- use profiler e testes antes de otimizar;
- versione o projeto com [[Git]] e documente a forma de exportação.

## Em uma frase

**Godot Engine é uma engine gratuita e de código aberto para criar jogos 2D e 3D, organizada em nodes e scenes e programável principalmente com GDScript, C# e extensões nativas.**

## Fontes

- [Introdução ao Godot Engine — documentação oficial](https://docs.godotengine.org/en/stable/about/introduction.html)
- [Visão geral dos conceitos principais — documentação oficial](https://docs.godotengine.org/en/stable/getting_started/introduction/key_concepts_overview.html)
- [Nodes e Scenes — documentação oficial](https://docs.godotengine.org/en/stable/getting_started/step_by_step/nodes_and_scenes.html)
- [Using signals — documentação oficial](https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html)
- [C# basics — documentação oficial](https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/c_sharp_basics.html)
- [Exportando projetos — documentação oficial](https://docs.godotengine.org/en/stable/tutorials/export/exporting_projects.html)
- [Networking — documentação oficial](https://docs.godotengine.org/en/stable/tutorials/networking/index.html)
