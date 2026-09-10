# O que é Unreal Engine?

**Unreal Engine** é uma engine (motor) para criar jogos e experiências interativas em 2D e 3D. Ela fornece uma grande parte da estrutura pronta: renderização de imagens, iluminação, física, áudio, animação, entrada do jogador, interface, ferramentas de edição e recursos para multiplayer.

Uma analogia simples: criar um jogo sem uma engine seria como construir um carro começando pelo motor, pelas rodas e por cada parafuso. A Unreal Engine já oferece muitos desses sistemas básicos, permitindo que a equipe se concentre nas regras, no conteúdo e na experiência do jogo.

A Unreal Engine é desenvolvida pela Epic Games. O programa principal de criação é o **Unreal Editor**, no qual o projeto é montado, testado e configurado.

## O que pode ser criado?

Com a Unreal Engine, é possível criar:

- jogos para computador, consoles e dispositivos móveis;
- experiências de realidade virtual e aumentada;
- visualizações arquitetônicas e produtos interativos;
- simulações e treinamentos;
- cinematics e ambientes 3D;
- aplicações que combinam gráficos em tempo real com dados externos.

## Engine, projeto e jogo

Esses termos não significam a mesma coisa:

- **engine:** o conjunto de tecnologia e ferramentas fornecido pela Unreal;
- **projeto:** os arquivos específicos de uma aplicação, incluindo código, mapas, configurações e assets;
- **jogo:** o produto executável criado a partir do projeto.

Um projeto costuma ter um arquivo `.uproject`, uma pasta `Content` para assets e módulos de código quando usa C++. O Content Browser do editor ajuda a encontrar e organizar esses assets.

## Actors e Components

Um **Actor** é um objeto que pode existir em um Level (nível ou mapa), como um personagem, uma câmera, uma luz, uma porta ou um item coletável. Em C++, a classe-base de um Actor é `AActor`.

Um **Component** é uma parte que acrescenta uma capacidade a um Actor. É como montar um carro:

- o carro é o Actor;
- o motor, as rodas, a câmera e o som são Components;
- cada componente acrescenta uma responsabilidade ao conjunto.

Exemplos comuns:

- `UStaticMeshComponent`: exibe uma malha 3D;
- `USkeletalMeshComponent`: exibe um personagem ou objeto com esqueleto;
- `UCameraComponent`: representa uma câmera;
- `UAudioComponent`: reproduz áudio;
- `UBoxComponent` e `USphereComponent`: ajudam com colisões e sobreposições;
- `UActorComponent`: adiciona comportamento sem precisar representar uma forma física.

O Actor usa um componente-raiz (*Root Component*) para definir sua transformação, como posição, rotação e escala. Components de cena podem ser ligados em uma hierarquia.

## Levels e Worlds

Um **Level** é um ambiente ou mapa que contém Actors. Pode representar uma fase, uma sala, uma cidade ou uma área do mundo do jogo.

Um **World** é o contexto maior que contém o Level em execução e outros elementos necessários para simular o jogo. Um projeto pode carregar níveis diferentes, dividir um mundo em partes e organizar o conteúdo de acordo com a necessidade.

A separação entre assets, níveis e código ajuda a equipe a trabalhar com mais segurança. Nomes claros e pastas coerentes no Content Browser facilitam a manutenção.

## Blueprints

**Blueprints** são um sistema visual de programação. Em vez de escrever todas as instruções, o desenvolvedor conecta nós que representam eventos, condições, chamadas de função e valores.

Por exemplo, um Blueprint pode representar o fluxo:

```text
Jogador entrou na área
        ↓
Verificar se possui a chave
        ↓
Abrir a porta ou mostrar uma mensagem
```

Blueprints não são apenas configurações visuais: eles podem conter lógica de gameplay. São úteis para prototipar, ajustar valores e permitir que designers trabalhem sem alterar todo o código.

Boas práticas:

- mantenha os gráficos pequenos e organizados;
- divida lógica grande em funções ou outros Blueprints;
- use nomes claros para variáveis e eventos;
- evite esconder regras críticas em muitos nós espalhados;
- exponha no Blueprint somente o que precisa ser ajustado no editor.

## C++ na Unreal

A Unreal Engine usa C++ para criar sistemas de gameplay, componentes, ferramentas e partes que precisam de mais controle ou desempenho. O código pode ser combinado com Blueprints: C++ fornece uma base reutilizável e Blueprints permitem ajustes rápidos.

Uma classe simples de Actor pode ser declarada assim:

```cpp
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Contador.generated.h"

UCLASS()
class MEUJOGO_API AContador : public AActor
{
    GENERATED_BODY()

public:
    AContador();

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Contador")
    int32 Pontos = 0;

    UFUNCTION(BlueprintCallable, Category = "Contador")
    void AdicionarPontos(int32 Quantidade);
};
```

Uma implementação possível:

```cpp
#include "Contador.h"

AContador::AContador()
{
    PrimaryActorTick.bCanEverTick = false;
}

void AContador::AdicionarPontos(int32 Quantidade)
{
    if (Quantidade > 0)
    {
        Pontos += Quantidade;
    }
}
```

Nesse exemplo:

- `UCLASS` informa à Unreal que a classe participa do sistema de reflexão;
- `UPROPERTY` permite que uma propriedade seja conhecida pelo editor e, quando indicado, por Blueprints;
- `UFUNCTION` pode expor uma função para ser chamada em um Blueprint;
- `AActor` indica que a classe é um Actor;
- `int32` é um tipo inteiro usado com frequência nos tipos da Unreal;
- `PrimaryActorTick.bCanEverTick = false` desativa atualizações por quadro quando elas não são necessárias.

As macros conectam o código C++ ao editor, à serialização e ao sistema de reflexão da engine. O nome `MEUJOGO_API` varia de acordo com o projeto.

## Ciclo de vida e Tick

Um Actor possui eventos que ajudam a organizar seu ciclo de vida. `BeginPlay` é usado quando ele começa a participar do jogo. `Tick` pode ser executado a cada quadro para atualizar o Actor.

```cpp
void AMeuActor::BeginPlay()
{
    Super::BeginPlay();
    // Preparação inicial do Actor.
}
```

Usar `Tick` para tudo pode consumir muitos recursos, principalmente quando existem centenas ou milhares de Actors. Quando possível, prefira eventos, timers, colisões ou outras chamadas feitas somente quando algo realmente acontece.

## Gameplay Framework

A Unreal oferece classes que organizam partes comuns de um jogo:

- `GameMode`: regras principais da partida; no multiplayer, existe no servidor;
- `GameState`: estado da partida que pode ser compartilhado;
- `PlayerController`: representa a intenção e a entrada de um jogador;
- `Pawn`: objeto controlável;
- `Character`: tipo de Pawn preparado para personagens com movimento;
- `HUD` e widgets: elementos de informação e interface apresentados ao jogador.

Essas classes não substituem o design do jogo. Elas oferecem pontos de extensão para que as regras sejam colocadas em locais previsíveis.

## Assets, materiais e iluminação

Um **asset** é um recurso usado pelo projeto, como uma malha 3D, textura, som, animação, material ou Blueprint.

Um **material** define como uma superfície aparece: cor, rugosidade, metalização, transparência e outras propriedades. Texturas fornecem imagens que podem alimentar partes desse material.

Para manter o projeto organizado:

- use nomes e pastas consistentes;
- evite duplicar assets sem necessidade;
- mantenha materiais reutilizáveis;
- controle tamanho e formato de texturas;
- descarte assets não utilizados;
- valide o resultado em diferentes níveis de qualidade e plataformas.

## Física, colisão e animação

Components podem representar colisões, corpos físicos, malhas estáticas e malhas esqueléticas. A engine usa essas informações para simular movimento, detectar sobreposições e reproduzir animações.

É importante diferenciar:

- colisão usada para bloquear movimento;
- sobreposição (*overlap*) usada para detectar que objetos entraram em uma área;
- simulação física usada para calcular forças e movimento.

Escolher colisores simples e ativar física somente quando necessária reduz o custo da simulação.

## Memória e objetos da Unreal

A Unreal usa C++, mas nem todo objeto deve ser gerenciado como um objeto C++ comum.

- para Actors, use as APIs da engine, como `SpawnActor` e `Destroy`;
- para objetos derivados de `UObject`, use as formas de criação e referências esperadas pelo sistema da Unreal;
- não use `delete` manualmente em um `UObject` controlado pela engine;
- para objetos C++ comuns, use RAII e tipos da biblioteca padrão;
- mantenha referências válidas e informe ao sistema da Unreal quais propriedades precisam ser rastreadas.

Misturar incorretamente gerenciamento manual de memória com o sistema da engine pode causar vazamentos, acessos inválidos e falhas difíceis de reproduzir.

## Multiplayer e replication

Em um jogo multiplayer, várias máquinas precisam enxergar uma versão coerente do mundo. A Unreal usa uma arquitetura cliente-servidor:

- o servidor mantém o estado autoritativo da partida;
- os clientes enviam intenções e recebem atualizações;
- **replication** sincroniza propriedades e chamadas entre servidor e clientes;
- RPCs (*Remote Procedure Calls*) permitem solicitar ações em outro contexto de rede.

Por exemplo, o cliente pode pedir para o servidor tentar disparar uma arma, mas o servidor deve validar munição, posição e regras antes de aceitar a ação.

Se houver possibilidade de multiplayer, planeje isso desde o início. Transformar um jogo single-player em multiplayer depois pode exigir mudanças profundas na arquitetura.

Boas práticas de rede:

- deixe regras importantes no servidor;
- nunca confie no cliente para decidir dano, dinheiro ou permissões;
- replique somente os dados necessários;
- evite enviar eventos a cada quadro sem necessidade;
- use RPCs com moderação;
- considere latência, perda de pacotes e reconexão;
- teste com condições de rede ruins.

## Mapa procedural

Um mapa procedural é gerado por regras ou algoritmos, em vez de ser totalmente desenhado à mão. Na Unreal, ele pode ser construído com C++, Blueprints ou sistemas específicos da engine.

Um algoritmo simples poderia:

1. escolher uma semente (*seed*);
2. percorrer uma grade;
3. decidir se cada célula é terreno, água ou obstáculo;
4. criar ou instanciar os elementos necessários;
5. validar se o jogador consegue atravessar o mapa.

O tema se conecta ao arquivo [[Mapa procedural]]. Para grandes quantidades de objetos, considere instancing, carregamento por partes e geração sob demanda para evitar desperdício de memória e tempo.

## Unreal Engine e Unity

Unreal e [[Unity Engine]] são engines usadas para criar jogos, mas possuem modelos e ferramentas diferentes:

- Unreal combina C++ com Blueprints; Unity usa C# como linguagem principal de scripts;
- Unreal chama seus objetos de cena de Actors; Unity usa GameObjects;
- as duas usam Components para acrescentar capacidades aos objetos;
- Unreal organiza ambientes em Levels; Unity trabalha com Scenes;
- as duas oferecem renderização, física, áudio, animação, editor e recursos de multiplayer.

Os conceitos são transferíveis, mas o código, a API e o fluxo de trabalho não são iguais. Estudar [[C++]] ajuda especialmente na programação nativa da Unreal; estudar [[C#]] ajuda nos scripts da Unity.

## Performance

Desempenho não significa somente fazer o código executar rápido. Também envolve memória, carregamento, rede, renderização e estabilidade do frame rate.

Boas práticas:

- meça com os profilers da engine antes de otimizar;
- desligue `Tick` quando um evento ou timer for suficiente;
- evite criar e destruir muitos Actors repetidamente; avalie object pooling;
- reutilize materiais, meshes e Components quando fizer sentido;
- carregue conteúdo por demanda em mundos grandes;
- limite dados replicados pela rede;
- teste em hardware semelhante ao que o público usará.

Otimização prematura pode deixar o código mais difícil sem resolver o gargalo real.

## Testes, versionamento e colaboração

Projetos de jogos também precisam de [[Testes]]. Além de testar funções e regras, é útil testar carregamento de níveis, colisões, salvamento, multiplayer e diferentes configurações de qualidade.

Use [[Git]] para versionar código e configurações. Assets binários grandes podem exigir uma estratégia própria, como Git LFS ou um sistema de controle de arquivos adequado à equipe. Combine regras para nomes, pastas e revisão de Blueprints para reduzir conflitos.

Princípios de [[SOLID]] e [[Design Pattern]] podem ajudar a organizar sistemas de gameplay, mas não devem ser aplicados mecanicamente. O design precisa continuar compreensível para a equipe que vai manter o projeto.

## Unreal Engine não é apenas o editor

O editor é a parte mais visível, mas um projeto também envolve:

- código e módulos C++;
- Blueprints e assets;
- configurações de plataforma;
- ferramentas de build e empacotamento;
- testes e automação;
- controle de versão;
- publicação e suporte após o lançamento.

Pensar no projeto inteiro evita que um protótipo funcione apenas na máquina do desenvolvedor e ajuda a preparar o jogo para distribuição.

## Em uma frase

**A Unreal Engine é uma plataforma completa para criar jogos e experiências interativas, combinando ferramentas visuais, C++, Blueprints e sistemas prontos de gráficos, física, áudio, animação e rede.**

## Fontes

- [Actors na Unreal Engine — Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/actors-in-unreal-engine)
- [Components na Unreal Engine — Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/components-in-unreal-engine)
- [Programação com C++ na Unreal Engine — Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-with-cplusplus-in-unreal-engine?lang=en-US)
- [Guia rápido de C++ na Unreal Engine — Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-cpp-quick-start?lang=en-US)
- [Visão geral de networking na Unreal Engine — Epic Developer Community](https://dev.epicgames.com/documentation/unreal-engine/networking-overview-for-unreal-engine?lang=en-US)
- [Blueprints — Epic Developer Community](https://dev.epicgames.com/documentation/en-us/unreal-engine/technical-guide-for-blueprints-visual-scripting-in-unreal-engine)
