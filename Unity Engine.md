# O que é Unity Engine?

**Unity Engine** é uma engine (*motor*) para criar jogos e aplicações interativas em 2D e 3D. Ela fornece ferramentas para renderização, física, áudio, animação, entrada do usuário, cenas, scripts e publicação para diferentes plataformas.

Uma analogia: criar um jogo do zero seria como construir um carro começando pelo motor, rodas e painel. A Unity já oferece muitos desses componentes básicos. A pessoa desenvolvedora combina as peças, cria as regras do jogo e define a experiência do jogador.

A Unity não é apenas uma [[Framework]] nem somente uma biblioteca. Ela reúne um editor visual, um runtime, sistemas de conteúdo e APIs para desenvolver e executar uma aplicação interativa.

## O que pode ser criado?

Com Unity Engine, é possível criar:

- jogos 2D;
- jogos 3D;
- experiências de realidade virtual e aumentada;
- simuladores;
- visualizações interativas;
- protótipos;
- aplicações para computador, celular, console e navegador, conforme o suporte e a configuração da versão usada.

O projeto define a plataforma de destino, os controles, os gráficos, o áudio, a lógica e os recursos necessários.

## Unity Editor e projeto

O **Unity Editor** é o ambiente visual onde o projeto é montado. Nele, a equipe pode:

- criar e editar cenas;
- posicionar objetos;
- adicionar componentes;
- configurar propriedades no Inspector;
- importar imagens, sons, modelos e scripts;
- testar em Play Mode;
- configurar o build.

Um projeto Unity normalmente contém assets, cenas, scripts, configurações e pacotes. As pastas geradas pelo editor podem ser recriadas e devem ser tratadas de maneira diferente dos arquivos-fonte.

## Conceitos fundamentais

### Scene

Uma **Scene** (cena) é um espaço ou estado da aplicação. Ela pode representar:

- uma fase do jogo;
- uma tela de menu;
- uma tela de carregamento;
- uma arena;
- uma cutscene;
- uma parte de um simulador.

Uma aplicação pode carregar cenas diferentes durante a execução. Planeje o que deve permanecer entre cenas, como configurações, áudio ou dados de sessão.

### GameObject

Um **GameObject** é o objeto básico presente em uma cena. Personagens, câmeras, luzes, itens, obstáculos e efeitos podem ser GameObjects.

Um GameObject vazio não possui comportamento útil por conta própria. Ele funciona como um recipiente que recebe componentes.

### Component

Um **Component** (componente) adiciona uma capacidade ao GameObject. Exemplos:

- `Transform`: posição, rotação e escala;
- `MeshRenderer`: desenha um modelo;
- `Camera`: observa a cena;
- `Light`: produz iluminação;
- `Collider`: representa uma forma para colisão;
- `Rigidbody`: participa da simulação física;
- script: implementa comportamento criado pela equipe.

Todo GameObject possui um `Transform`. Os outros componentes podem ser combinados conforme a necessidade.

Essa composição é parecida com montar um personagem usando peças: o mesmo GameObject pode receber componentes de renderização, colisão, áudio e comportamento sem precisar ser uma classe gigante.

### Inspector

O **Inspector** exibe e permite editar as propriedades do objeto selecionado. Valores marcados com `[SerializeField]` podem ser configurados no editor sem ficarem públicos para todo o código:

```csharp
using UnityEngine;

public class Porta : MonoBehaviour
{
    [SerializeField] private float velocidade = 2f;
}
```

O campo fica configurável no Inspector, enquanto continua `private` para outras classes. Isso ajuda a separar configuração de implementação.

### Prefab

Um **Prefab** é um modelo reutilizável de GameObject, com seus componentes, propriedades e filhos.

Em vez de copiar manualmente dez inimigos, crie um Prefab de inimigo e coloque instâncias dele nas cenas. Alterações feitas no Prefab podem ser refletidas nas instâncias, respeitando eventuais overrides.

Prefabs são úteis para:

- personagens;
- inimigos;
- projéteis;
- itens coletáveis;
- efeitos;
- elementos de interface;
- partes repetidas do cenário.

## Scripts e MonoBehaviour

Os scripts da Unity normalmente são escritos em C#. Um script que precisa ser anexado a um GameObject geralmente herda de `MonoBehaviour`:

```csharp
using UnityEngine;

public class GirarObjeto : MonoBehaviour
{
    [SerializeField] private float grausPorSegundo = 90f;

    private void Update()
    {
        transform.Rotate(
            0f,
            grausPorSegundo * Time.deltaTime,
            0f
        );
    }
}
```

Depois de salvar o script, ele pode ser adicionado como componente a um GameObject. A cada frame, `Update` gira o objeto usando `Time.deltaTime`, fazendo a velocidade ser menos dependente da taxa de frames.

`MonoBehaviour` é adequado para comportamentos ligados a GameObjects. Objetos de dados que não precisam existir na cena podem usar outras formas, como `ScriptableObject` ou classes C# comuns.

## Ciclo de vida

A Unity chama métodos conhecidos em momentos diferentes:

| Método | Uso comum |
|---|---|
| `Awake` | Inicializar referências quando o script é carregado. |
| `OnEnable` | Reagir quando o componente é ativado. |
| `Start` | Preparar o comportamento antes do primeiro `Update`. |
| `Update` | Processar lógica por frame. |
| `FixedUpdate` | Trabalhar com física em intervalo fixo. |
| `LateUpdate` | Executar depois dos `Update`, como ajustar uma câmera. |
| `OnDisable` | Limpar inscrições quando o componente é desativado. |
| `OnDestroy` | Liberar recursos quando o objeto é destruído. |

Não crie um `Update` vazio em todo script. Se o comportamento não precisa ser executado a cada frame, use eventos, callbacks ou métodos chamados quando algo realmente acontecer.

## Update e FixedUpdate

`Update` é chamado a cada frame renderizado. Ele costuma ser usado para:

- ler entrada;
- atualizar uma animação lógica;
- verificar temporizadores;
- controlar comportamentos que dependem da imagem atual.

`FixedUpdate` usa o intervalo fixo do sistema de física. Ele é indicado para aplicar forças e trabalhar com `Rigidbody`.

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class Impulso : MonoBehaviour
{
    [SerializeField] private float forca = 5f;
    private Rigidbody corpo;

    private void Awake()
    {
        corpo = GetComponent<Rigidbody>();
    }

    private void FixedUpdate()
    {
        corpo.AddForce(Vector3.up * forca);
    }
}
```

`[RequireComponent]` informa que o GameObject precisa ter um `Rigidbody`. Isso reduz o risco de executar o script sem a dependência necessária.

## Física e colisões

O sistema de física simula movimentos e interações. Os componentes mais comuns são:

- `Collider`: define a forma usada nas colisões;
- `Rigidbody`: permite simulação de massa, gravidade e forças;
- `Is Trigger`: detecta passagem ou sobreposição sem bloquear fisicamente o objeto;
- `Physics Material`: controla atrito e elasticidade.

Uma colisão pode chamar métodos como `OnCollisionEnter`. Um trigger pode chamar `OnTriggerEnter`.

Não misture alterações diretas de `transform` com física sem entender o efeito. Mover um objeto controlado por Rigidbody diretamente pode produzir resultados inesperados na simulação.

## Renderização

Renderização (*rendering*) é o processo de transformar modelos, materiais, luzes e câmeras em pixels na tela.

A Unity oferece pipelines de renderização com objetivos diferentes, como:

- pipeline integrado;
- URP, voltado a boa flexibilidade e vários dispositivos;
- HDRP, voltado a gráficos mais avançados e hardware compatível.

Escolha o pipeline no início do projeto quando possível. Migrar depois pode exigir ajustes em materiais, shaders e efeitos.

Qualidade visual não depende somente do pipeline. Modelos, texturas, iluminação, shaders, resolução, pós-processamento e hardware também influenciam o resultado.

## Assets e Package Manager

**Assets** são os recursos usados pelo projeto, como:

- scripts;
- imagens e texturas;
- modelos 3D;
- materiais;
- animações;
- músicas e efeitos sonoros;
- fontes;
- cenas;
- prefabs.

O **Package Manager** permite adicionar pacotes e recursos à instalação. Mantenha as versões registradas e evite adicionar pacotes sem avaliar manutenção, licença, dependências e impacto no build.

## Dados com ScriptableObject

`ScriptableObject` é uma classe de dados que pode ser salva como asset e compartilhada por componentes.

Ele é útil para configurações, itens, armas, personagens e parâmetros que não precisam ser duplicados em cada GameObject:

```csharp
using UnityEngine;

[CreateAssetMenu(menuName = "Jogo/Item")]
public class ItemConfig : ScriptableObject
{
    public string nome;
    public int valor;
}
```

Uma instância desse asset pode ser referenciada por vários objetos. Tome cuidado ao alterar dados em runtime: um asset compartilhado pode afetar vários consumidores.

## Comunicação com backend

Um jogo conectado pode conversar com um [[Backend]] para login, inventário, partidas, pagamentos ou sincronização.

```text
Unity -> [[Requisição]] HTTPS -> Backend -> banco ou serviço externo
```

Os dados podem ser enviados em [[JSON]], mas o cliente não deve ser tratado como uma fonte confiável. Valide regras, permissões e valores no backend.

Nunca coloque uma chave secreta permanente dentro do build do jogo. Um arquivo distribuído ao jogador pode ser examinado. Use autenticação adequada, tokens com escopo limitado e comunicação protegida por TLS.

## Arquitetura do código

Um projeto pequeno pode começar com scripts ligados a GameObjects. Conforme cresce, evite colocar toda a lógica em um único `MonoBehaviour`.

Separe, quando fizer sentido:

- apresentação e animação;
- entrada do jogador;
- regras do jogo;
- dados e configurações;
- acesso à rede;
- persistência local;
- áudio;
- interface.

Um `MonoBehaviour` pode coordenar o objeto sem conter todas as regras. Classes C# comuns são mais fáceis de testar quando não precisam do editor ou de uma cena para existir.

Padrões como composição, eventos, State, Strategy e Object Pooling podem ajudar, mas devem ser usados para resolver problemas reais. Consulte [[Design Pattern]] e [[Programação procedural]] para comparar formas de organizar o código.

## Desempenho

Uma aplicação em tempo real precisa executar seu trabalho dentro do tempo disponível para cada frame. Se um frame demora demais, podem aparecer travamentos ou queda de FPS.

Boas práticas:

- use o Profiler para medir antes de otimizar;
- evite buscas repetidas e trabalho pesado em `Update`;
- guarde referências em vez de procurar objetos continuamente;
- reduza alocações desnecessárias durante o jogo;
- use object pooling para objetos criados e destruídos com muita frequência;
- carregue cenas e assets grandes de forma planejada;
- comprima e dimensione texturas conforme o dispositivo;
- use LOD (*Level of Detail*) quando fizer sentido;
- teste em dispositivos reais, não somente no editor;
- observe CPU, GPU, memória, carregamento e rede separadamente.

O Profiler ajuda a identificar onde o tempo está sendo gasto. Otimizar sem medir pode apenas trocar um problema por outro.

## Persistência e versionamento

Use [[Git]] para versionar scripts, cenas, prefabs, configurações e assets importantes. Como muitos arquivos Unity são binários ou têm referências, configure a estratégia de versionamento antes de o projeto crescer.

Boas práticas:

- não versionar pastas geradas que podem ser recriadas;
- manter o projeto em uma versão conhecida da Unity;
- documentar pacotes e dependências;
- evitar renomear ou mover assets fora do editor sem entender as referências;
- usar Git LFS quando o tamanho e o fluxo do repositório justificarem;
- fazer commits pequenos e descritivos;
- testar se outra pessoa consegue clonar e abrir o projeto.

## Testes

Há diferentes níveis de [[Testes]] em um projeto Unity:

- **Edit Mode:** testa classes e lógica sem executar a cena;
- **Play Mode:** testa comportamentos com a Unity em execução;
- **integração:** verifica comunicação entre componentes, cenas ou serviços;
- **build:** confirma o comportamento na plataforma de destino.

Regras de pontuação, cálculo de dano e validações podem ser classes C# independentes e testadas sem criar uma cena inteira. Testes visuais e de input precisam de estratégias adicionais.

## Build e CI/CD

O **build** transforma o projeto em uma aplicação executável para uma plataforma. A configuração inclui cena inicial, plataforma, arquitetura, qualidade, permissões e recursos incluídos.

O [[CI-CD]] pode automatizar:

- abrir o projeto em modo de build;
- executar testes;
- verificar scripts;
- gerar um build de desenvolvimento;
- armazenar artefatos;
- publicar versões de teste.

Builds de Unity podem exigir licença, memória, tempo e arquivos grandes. Planeje caches, armazenamento e credenciais do ambiente automatizado sem colocá-las no repositório.

## Unity é uma engine, não o jogo

A Unity fornece o motor e as ferramentas, mas não cria automaticamente:

- uma boa mecânica de jogo;
- uma arquitetura adequada;
- arte, áudio ou roteiro;
- regras de negócio corretas;
- segurança de um backend;
- testes suficientes;
- desempenho em todos os dispositivos.

A engine acelera o trabalho, mas ainda é necessário projetar, implementar, testar e otimizar o produto.

## Boas práticas

1. **Aprenda a relação entre Scene, GameObject e Component.** Ela é a base do editor.
2. **Prefira composição.** Combine componentes pequenos em vez de criar uma classe enorme.
3. **Use Prefabs para objetos reutilizáveis.** Isso reduz cópias difíceis de manter.
4. **Use `[SerializeField] private` para configurações editáveis.** Exponha somente o que precisa aparecer no Inspector.
5. **Separe lógica de jogo e apresentação.** Regras independentes são mais fáceis de testar.
6. **Use `Update` e `FixedUpdate` no lugar correto.** Física e renderização possuem ritmos diferentes.
7. **Não dependa de buscas globais frequentes.** Referencie dependências de maneira explícita.
8. **Mensure com o Profiler.** Decisões de desempenho devem usar dados reais.
9. **Teste no dispositivo de destino.** O editor pode apresentar resultados diferentes.
10. **Proteja o cliente online.** O jogador pode modificar o build e enviar dados falsos.
11. **Versione o projeto com cuidado.** Cenas e assets podem ter conflitos e referências complexas.
12. **Mantenha pacotes e engine atualizados com planejamento.** Atualizações podem exigir migração e testes.
13. **Não coloque segredos no cliente.** Tudo distribuído ao jogador pode ser inspecionado.
14. **Faça backups dos assets.** Recriar arte, cenas e configurações pode ser caro.

## Resumo

Unity Engine é uma engine para criar jogos e aplicações interativas em 2D e 3D. O projeto é construído com Scenes, GameObjects, Components, Prefabs, assets e scripts C#. A Unity cuida de muitos detalhes de renderização, física, áudio e execução, enquanto a equipe cria as regras e a experiência. Para projetos saudáveis, organize componentes, separe responsabilidades, teste, versione e meça o desempenho no dispositivo real.

### Veja também

- [[Framework]]
- [[Java]]
- [[Backend]]
- [[Requisição]]
- [[JSON]]
- [[TLS]]
- [[Git]]
- [[Testes]]
- [[CI-CD]]
- [[Design Pattern]]
- [[Programação procedural]]

### Referências oficiais

- [Conceitos fundamentais da Unity](https://docs.unity3d.com/Manual/key-concepts.html)
- [GameObjects e componentes](https://docs.unity3d.com/Manual/GameObjects.html)
- [Introdução a Prefabs](https://docs.unity3d.com/Manual/prefabs-introduction.html)
- [MonoBehaviour](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html)
- [Update](https://docs.unity3d.com/ScriptReference/MonoBehaviour.Update.html)
- [FixedUpdate](https://docs.unity3d.com/ScriptReference/MonoBehaviour.FixedUpdate.html)
- [Profiler](https://docs.unity3d.com/Manual/ProfilerWindow.html)
