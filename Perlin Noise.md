# O que é Perlin Noise?

**Perlin Noise** (ruído de Perlin) é um algoritmo que produz valores pseudoaleatórios com variação suave e contínua. Ele é muito usado em computação gráfica, jogos e geração procedural para criar terrenos, nuvens, texturas, ondas, biomas e movimentos naturais.

O nome “ruído” pode causar confusão. Não é um ruído totalmente aleatório, como pontos escolhidos sem relação uns com os outros. No Perlin Noise, pontos próximos costumam ter valores parecidos, criando transições suaves.

Uma analogia: imagine uma paisagem feita de colinas. Uma colina não passa instantaneamente de muito alta para muito baixa; a altura muda aos poucos. O Perlin Noise produz justamente esse tipo de variação controlada.

## Para que serve?

Ele pode ser usado para:

- criar alturas de terreno;
- separar água, areia, terra e montanhas;
- gerar biomas;
- produzir nuvens, fumaça e mármore;
- variar a posição, cor ou tamanho de objetos;
- animar movimentos orgânicos;
- criar mapas procedurais na [[Unity Engine]], [[Unreal Engine]] e [[Godot Engine]].

Perlin Noise não cria um jogo completo sozinho. Ele fornece um campo de valores; as regras do jogo transformam esses valores em objetos e comportamentos.

## Como pensar no resultado

Em duas dimensões, podemos imaginar uma função:

```text
valor = noise(x, y)
```

Para cada posição `(x, y)`, o algoritmo devolve um valor. Em geral, esse valor é convertido para uma faixa conveniente, como `0` a `1`:

- perto de `0`: valor baixo;
- perto de `0.5`: valor intermediário;
- perto de `1`: valor alto.

Um mapa de alturas pode interpretar esses valores assim:

```text
0.00 até 0.30 → água
0.30 até 0.50 → areia ou planície
0.50 até 0.75 → terra
0.75 até 1.00 → montanha
```

Esses limites são uma decisão do projeto, não uma regra do algoritmo.

## Por que ele é suave?

O Perlin Noise trabalha sobre uma grade. De forma simplificada, o algoritmo:

1. divide o espaço em células;
2. associa um gradiente pseudoaleatório aos cantos da célula;
3. calcula como o ponto se relaciona com os gradientes próximos;
4. faz uma interpolação suave entre os cantos;
5. devolve o valor resultante.

O gradiente é uma direção, e não apenas um número aleatório. O produto escalar entre essa direção e o vetor até o ponto ajuda a gerar transições naturais.

Na versão conhecida como **Improved Perlin Noise**, uma função de suavização (*fade function*) reduz mudanças bruscas nas bordas das células. Uma forma clássica dessa função é:

```text
fade(t) = 6t⁵ - 15t⁴ + 10t³
```

O resultado continua sendo determinístico: os mesmos pontos e a mesma configuração produzem os mesmos valores.

## Seed

A **seed** é a semente usada para escolher a sequência pseudoaleatória. Pense nela como o número inicial de um baralho embaralhado:

- mesma seed → mesmo mapa;
- seed diferente → mapa diferente;
- guardar a seed → possibilidade de reproduzir o mapa depois.

Usar uma seed fixa durante o desenvolvimento facilita encontrar e corrigir bugs. No jogo final, a seed pode ser escolhida pelo jogador ou gerada a partir de uma partida.

Não confunda seed com segurança criptográfica. Perlin Noise não deve ser usado para senhas, tokens ou qualquer valor que precise ser imprevisível contra um atacante.

## Escala e frequência

A escala define o tamanho das características produzidas:

- variações com baixa frequência formam grandes colinas e regiões amplas;
- variações com alta frequência formam detalhes menores e mais numerosos.

Um modo comum de calcular a posição de consulta é:

```text
amostra_x = x * frequência + deslocamento_x
amostra_y = y * frequência + deslocamento_y
```

Se a frequência for muito baixa, o mapa pode parecer quase plano. Se for muito alta, haverá muitas mudanças pequenas e o terreno poderá parecer granulado.

O nome `noiseScale` é usado por muitas implementações, mas algumas tratam esse valor como uma escala que deve ser dividida, e outras como uma frequência que deve ser multiplicada. Verifique sempre a convenção da biblioteca usada.

## Octaves e fractal Brownian motion

Uma única camada de Perlin Noise costuma produzir um formato suave demais. Para adicionar detalhes, combinamos várias camadas, chamadas **octaves**:

```text
resultado = 0
frequência = frequência_inicial
amplitude = 1

para cada octave:
    resultado += noise(x * frequência, y * frequência) * amplitude
    frequência *= lacunarity
    amplitude *= persistence
```

Os principais parâmetros são:

- **octaves:** quantidade de camadas;
- **frequency:** distância entre variações em uma camada;
- **lacunarity:** quanto a frequência aumenta entre camadas;
- **amplitude:** força de uma camada;
- **persistence:** quanto a amplitude diminui entre camadas.

As primeiras camadas definem as formas grandes. As últimas acrescentam detalhes menores. Depois da soma, é importante normalizar o resultado para a faixa esperada.

## Exemplo em C#

Uma engine pode fornecer uma função pronta. Na Unity, por exemplo, `Mathf.PerlinNoise` pode ser consultado para obter um valor de uma posição:

```csharp
float frequencia = 0.02f;
float deslocamentoX = seed * 17.13f;
float deslocamentoY = seed * 31.71f;

float valor = Mathf.PerlinNoise(
    x * frequencia + deslocamentoX,
    y * frequencia + deslocamentoY);

if (valor < 0.35f)
{
    // Criar água.
}
else if (valor < 0.70f)
{
    // Criar terra.
}
else
{
    // Criar montanha.
}
```

O deslocamento derivado da seed muda o padrão sem mudar a escala. Os limites devem ser ajustados observando a distribuição real dos valores.

## Exemplo de mapa de alturas

Um gerador de terreno pode seguir este fluxo:

```text
para cada célula do mapa:
    valor = noise(célula_x * frequência, célula_y * frequência)
    altura = valor * altura_máxima

    se valor < limite_água:
        tipo = água
    senão se valor < limite_terra:
        tipo = terra
    senão:
        tipo = montanha
```

Depois, a engine transforma a altura em uma malha, tiles ou instâncias de objetos. O Perlin Noise define o padrão geral; regras adicionais podem garantir rios, caminhos, áreas de início e locais acessíveis.

## Domínios 1D, 2D e 3D

O algoritmo pode ser usado em diferentes dimensões:

- **1D:** variar a altura de uma linha ou o movimento de um objeto;
- **2D:** gerar mapas, texturas e terrenos vistos de cima;
- **3D:** criar volumes, nuvens, cavernas, efeitos e texturas procedurais.

Também é possível usar o tempo como uma dimensão adicional para gerar animação. Porém, mover o tempo de qualquer forma pode fazer o padrão “escorregar”. Para animações naturais, use coordenadas e velocidade com uma intenção clara.

## Perlin Noise não é qualquer ruído

Existem outros tipos de ruído procedural:

- **white noise:** valores independentes, com aparência mais granulada;
- **Simplex Noise:** alternativa criada para reduzir alguns custos e problemas do Perlin em dimensões maiores;
- **Worley ou cellular noise:** cria padrões baseados na distância até pontos, úteis para células e pedras;
- **value noise:** interpola valores definidos nos pontos da grade;
- **fractal noise:** combina camadas de ruído, como no uso de octaves.

Não existe um algoritmo melhor para todas as situações. A escolha depende do visual, do custo e do tipo de controle necessário.

## Limitações

Perlin Noise tem algumas limitações:

- não garante que o mapa seja divertido ou jogável;
- não entende regras como “deve existir um caminho entre dois pontos”;
- pode apresentar padrões direcionais ou repetição quando usado em excesso;
- várias octaves aumentam o custo de processamento;
- valores próximos não significam necessariamente objetos conectados;
- uma implementação diferente pode produzir faixa e distribuição de valores diferentes.

Por isso, um gerador de mundo normalmente combina ruído com regras, validações, filtros, estruturas de dados e algoritmos de caminho.

## Boas práticas para mapas procedurais

- use uma seed reproduzível durante o desenvolvimento;
- separe geração, regras de bioma e renderização;
- normalize o resultado antes de aplicar limites;
- documente frequência, octaves, lacunarity e persistence;
- não gere o mapa inteiro a cada quadro;
- use chunks ou carregamento por partes em mundos grandes;
- reutilize objetos e considere instancing;
- valide caminhos, áreas inacessíveis e posições de nascimento;
- teste seeds extremas e mapas pequenos;
- salve a seed e os parâmetros quando o mundo precisar ser reproduzido;
- meça o custo antes de adicionar mais octaves ou detalhes.

## Uso nas engines

### Unity

A [[Unity Engine]] oferece `Mathf.PerlinNoise` para consultas de ruído em scripts [[C#]]. O resultado pode alimentar uma textura, uma altura de terreno ou uma regra de geração.

### Unreal Engine

Na [[Unreal Engine]], Perlin Noise pode ser usado em C++, materiais, ferramentas de geração e sistemas de conteúdo procedural. Em projetos que precisam de muitos objetos, é importante considerar instancing, carregamento por partes e custo de atualização.

### Godot

No [[Godot Engine]], o ruído pode ser usado em GDScript, C# ou extensões [[C++]]. Classes e recursos de ruído, como `FastNoiseLite`, podem fornecer Perlin e outros tipos de noise para gerar terrenos, texturas e efeitos.

## Relação com mapa procedural

Perlin Noise é uma ferramenta dentro da geração de [[Mapa procedural]], não um substituto para ela. Um mapa completo também precisa decidir:

- onde ficam os biomas;
- como regiões se conectam;
- onde colocar recursos e inimigos;
- como garantir caminhos jogáveis;
- como salvar e carregar o mundo;
- como sincronizar o resultado no multiplayer.

Em multiplayer, todos os participantes precisam receber ou reconstruir o mesmo mundo. Uma estratégia comum é compartilhar a seed e os parâmetros, desde que a implementação seja determinística em todas as máquinas.

## Em uma frase

**Perlin Noise é um algoritmo determinístico de variação suave que transforma coordenadas em valores pseudoaleatórios coerentes, sendo muito útil para criar padrões naturais e conteúdo procedural.**

## Fontes

- [Noise and Turbulence — Ken Perlin, NYU](https://cs.nyu.edu/~perlin/doc/oscar)
- [Implementação de Noise — Ken Perlin, NYU](https://mrl.cs.nyu.edu/projects/texture/noise.html)
- [Improving Noise — Ken Perlin](https://mrl.cs.nyu.edu/~perlin/paper445.pdf)
- [Mathf.PerlinNoise — Unity](https://docs.unity3d.com/ScriptReference/Mathf.PerlinNoise.html)
- [FastNoiseLite — Godot](https://docs.godotengine.org/en/stable/classes/class_fastnoiselite.html)
