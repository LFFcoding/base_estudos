# O que é um mapa procedural na Unity?

Um **mapa procedural** (*procedural map*) é um cenário criado automaticamente por regras, algoritmos e dados de entrada, em vez de ser desenhado inteiramente à mão.

Na [[Unity Engine]], um gerador pode criar terrenos, salas, cavernas, estradas, rios, inimigos e itens durante o desenvolvimento ou enquanto o jogo está sendo executado.

Uma analogia é um jogo de montar com regras: em vez de colocar cada peça manualmente, você informa o tamanho, as regras e uma semente. O gerador monta o cenário seguindo essas instruções.

## Mapa fixo e mapa procedural

| Mapa fixo | Mapa procedural |
|---|---|
| É desenhado e ajustado manualmente. | É criado por código ou regras. |
| Permite controle artístico muito detalhado. | Pode gerar muitas variações. |
| O resultado é conhecido antecipadamente. | O resultado pode mudar conforme a seed. |
| Pode exigir muito trabalho para criar fases grandes. | Pode economizar trabalho repetitivo. |
| Facilita testar uma fase específica. | Exige validação para evitar mapas impossíveis. |

Os dois estilos podem ser combinados. Uma equipe pode gerar a estrutura inicial e depois ajustar áreas importantes manualmente.

## A ideia principal: regras, não sorte pura

Geração procedural não é apenas escolher posições aleatórias. Um bom gerador possui regras:

- onde o jogador pode começar;
- onde pode existir uma saída;
- quais regiões podem ser conectadas;
- onde inimigos podem aparecer;
- que distância deve existir entre itens;
- quais áreas são seguras ou perigosas;
- qual quantidade de obstáculos é aceitável.

A aleatoriedade cria variedade, mas as regras mantêm o mapa jogável.

## Seed

Uma **seed** é um valor usado para iniciar o gerador de números pseudoaleatórios.

```text
seed 42 -> mapa A
seed 42 -> mapa A novamente
seed 99 -> mapa B
```

Usar a mesma seed deve produzir o mesmo resultado, desde que o algoritmo, a versão do jogo e as entradas sejam iguais.

Isso é útil para:

- reproduzir um bug;
- compartilhar um mapa;
- testar mudanças;
- jogar uma fase diária;
- sincronizar um cenário em uma partida;
- gerar o mapa no servidor e reconstruí-lo no cliente.

Guarde a seed junto com a versão do gerador. Se o algoritmo mudar, a mesma seed pode deixar de produzir o mapa antigo.

## Representação lógica

Antes de criar objetos visuais, represente o mapa como dados. Para um mapa em grade, cada célula pode ter um tipo:

```text
##########
#..#.....#
#..#..E..#
#..S.....#
##########
```

Por exemplo:

- `#`: parede;
- `.`: chão;
- `S`: início;
- `E`: saída.

Essa matriz é a **camada lógica**. Ela pode ser validada, salva e testada sem instanciar nenhum GameObject.

Depois, outra etapa transforma cada símbolo em uma imagem, tile ou prefab.

```text
regras -> matriz lógica -> validação -> renderização -> decoração
```

Separar essas etapas deixa o gerador mais fácil de testar e evita misturar regra de mapa com detalhes visuais.

## Geração em grade

Uma abordagem simples usa duas dimensões, `largura` e `altura`, e percorre cada célula:

```csharp
for (int y = 0; y < altura; y++)
{
    for (int x = 0; x < largura; x++)
    {
        mapa[x, y] = escolherTipo(x, y);
    }
}
```

`escolherTipo` pode consultar uma regra, ruído, distância do centro ou vizinhos. O resultado não precisa criar objetos imediatamente.

## Perlin Noise

**Perlin Noise** é um ruído contínuo. Ao contrário de sortear cada célula sem relação com as outras, ele produz valores que mudam gradualmente.

Isso pode criar regiões de montanha, planície, água e floresta com transições mais naturais:

```text
valor baixo  -> água
valor médio  -> planície
valor alto   -> montanha
```

Na Unity, `Mathf.PerlinNoise(x, y)` retorna um valor usado para amostrar um plano de ruído. Os valores podem servir como base para heightmaps, terrenos, texturas e mapas.

O `scale` controla o tamanho das regiões:

- escala pequena: mudanças mais rápidas e regiões menores;
- escala grande: mudanças mais suaves e regiões maiores.

## Exemplo de mapa com seed e ruído

O exemplo cria uma matriz de chão e parede. Ele usa `System.Random` para gerar o deslocamento do ruído, mas deixa a amostragem do Perlin Noise contínua:

```csharp
using System;
using UnityEngine;

public class GeradorDeMapa : MonoBehaviour
{
    [SerializeField] private int largura = 50;
    [SerializeField] private int altura = 30;
    [SerializeField] private int seed = 42;
    [SerializeField] private float escala = 12f;
    [SerializeField] private float limiteDeChao = 0.45f;

    private bool[,] mapa;

    public void Gerar()
    {
        if (largura <= 0 || altura <= 0 || escala <= 0)
        {
            throw new ArgumentException("Dimensões e escala inválidas");
        }

        mapa = new bool[largura, altura];
        var random = new System.Random(seed);
        int deslocamentoX = random.Next(-100_000, 100_000);
        int deslocamentoY = random.Next(-100_000, 100_000);

        for (int y = 0; y < altura; y++)
        {
            for (int x = 0; x < largura; x++)
            {
                float amostra = Mathf.PerlinNoise(
                    (x + deslocamentoX) / escala,
                    (y + deslocamentoY) / escala
                );

                mapa[x, y] = amostra >= limiteDeChao;
            }
        }
    }
}
```

Nesse exemplo, `true` pode significar chão e `false` pode significar parede. O gerador ainda precisa garantir início, saída, conectividade e visualização.

Uma seed determinística depende também da ordem dos sorteios. Se uma nova chamada aleatória for adicionada no meio do algoritmo, todos os resultados seguintes podem mudar.

## Criar o mapa visual

Depois de construir e validar a matriz, existem duas opções comuns:

### Tilemap

Em mapas 2D, um **Tilemap** pode representar células usando tiles. Ele costuma ser adequado para mapas grandes porque foi criado para organizar conteúdo em grade.

### Prefabs

Em mapas 2D ou 3D, é possível instanciar **Prefabs** para paredes, árvores, salas, objetos e decorações:

```csharp
private void CriarBloco(GameObject prefab, Vector3 posicao)
{
    Instantiate(prefab, posicao, Quaternion.identity, transform);
}
```

Usar Prefabs evita montar cada objeto do zero e permite alterar o visual sem mudar a regra de geração. Para muitos objetos, avalie batching, pooling e a quantidade de GameObjects criados.

## Exemplo de renderização da matriz

```csharp
[SerializeField] private GameObject prefabChao;
[SerializeField] private GameObject prefabParede;

private void Renderizar()
{
    for (int y = 0; y < altura; y++)
    {
        for (int x = 0; x < largura; x++)
        {
            GameObject prefab = mapa[x, y]
                ? prefabChao
                : prefabParede;

            Vector3 posicao = new Vector3(x, 0f, y);
            Instantiate(prefab, posicao, Quaternion.identity, transform);
        }
    }
}
```

O método deve ser chamado depois de `Gerar` e `Validar`. Não misture a ordem: renderizar antes de validar pode criar objetos para um mapa que será descartado.

## Algoritmos comuns

### Random walk

Um ponto começa em uma posição e caminha aleatoriamente, marcando células como chão. É simples e pode criar cavernas ou caminhos orgânicos.

Risco: o caminho pode não cobrir a área desejada ou pode formar regiões desconectadas. Defina limite de passos e valide o resultado.

### Cellular automata

Começa com uma grade aleatória e aplica regras baseadas nos vizinhos. Por exemplo, uma célula com muitas paredes ao redor vira parede.

É comum para gerar cavernas, mas exige cuidado com ilhas isoladas e áreas sem passagem.

### BSP

**Binary Space Partitioning** divide o espaço em partes menores e cria salas e corredores. É útil para masmorras organizadas.

O algoritmo pode controlar tamanho mínimo, distância entre salas e conexão entre regiões.

### Voronoi

Divide o mapa em regiões influenciadas por pontos. Pode ajudar a criar biomas, territórios ou áreas de influência.

### Wave Function Collapse

Usa regras de vizinhança e peças permitidas para montar padrões compatíveis. Pode criar mapas visualmente variados, mas as restrições precisam ser bem definidas.

### Ruído e heightmap

Combina ruídos para produzir alturas e biomas. Em terrenos 3D, a altura pode definir montanhas, vales e água.

Não existe um algoritmo universalmente melhor. Escolha de acordo com o tipo de mapa e o grau de controle necessário.

## Conectividade

Um mapa bonito pode ser impossível de jogar se o início não tiver caminho até a saída.

Depois de gerar a matriz, faça uma busca, como BFS ou DFS, a partir do início:

```text
início -> visitar vizinhos permitidos -> continuar
                              ↓
                 saída foi alcançada?
```

Validações possíveis:

- início e saída existem;
- há um caminho entre eles;
- uma porcentagem mínima do mapa é acessível;
- salas importantes estão conectadas;
- o jogador não nasce dentro de uma parede;
- itens obrigatórios podem ser alcançados;
- áreas perigosas não aparecem imediatamente no início;
- não há corredores estreitos demais para o personagem.

Se o mapa falhar, você pode regenerar com outra seed, corrigir a matriz ou rejeitar a geração. Não espere o jogador descobrir a falha durante a partida.

## Regras de vizinhança

O tipo de uma célula pode depender dos vizinhos:

```text
uma parede deve ter pelo menos dois vizinhos de parede
um rio não deve terminar no meio de uma montanha
uma estrada deve continuar conectada
```

Essas regras ajudam a evitar padrões estranhos. Porém, regras demais podem tornar a geração lenta ou impossível. Comece com poucas regras e aumente a complexidade somente quando houver um problema concreto.

## Separar geração, validação e renderização

Uma estrutura saudável é:

```text
GeradorDeDados
    ↓
ValidadorDoMapa
    ↓
RenderizadorDeMapa
    ↓
DecoradorDeMapa
```

- **Gerador:** cria a matriz e os dados;
- **Validador:** verifica regras e conectividade;
- **Renderizador:** cria tiles ou GameObjects;
- **Decorador:** adiciona árvores, inimigos, itens e efeitos.

Essa separação permite testar o algoritmo sem depender da cena e trocar o visual sem reescrever a lógica.

## Configurações com ScriptableObject

Parâmetros como largura, altura, seed, limites, prefabs e regras podem ficar em um `ScriptableObject`.

Isso permite criar configurações como:

```text
MapaFloresta.asset
MapaCaverna.asset
MapaDeserto.asset
```

O código usa a configuração sem deixar todos os valores fixos. Como um `ScriptableObject` é um asset compartilhável, não altere sua configuração global em runtime sem saber que outros mapas podem usar a mesma referência.

## Desempenho

Gerar um mapa grande pode consumir CPU, memória e tempo de carregamento.

Boas práticas:

- gere e valide a matriz antes de criar objetos;
- evite um GameObject para cada célula quando um Tilemap resolver o problema;
- agrupe objetos em regiões ou chunks;
- carregue áreas próximas e descarregue áreas distantes;
- use object pooling para inimigos, projéteis e efeitos repetidos;
- não execute uma geração pesada dentro de cada `Update`;
- use coroutines ou jobs quando o fluxo permitir;
- mantenha a seed e os parâmetros para reproduzir o cenário;
- meça com o Profiler da [[Unity Engine]];
- teste em dispositivos com menor desempenho.

Se o mapa for infinito, divida-o em chunks. Gere somente a área próxima do jogador e use a posição do chunk mais a seed global para obter resultados reproduzíveis.

## Mapas infinitos e chunks

Um mapa infinito não precisa existir inteiro na memória:

```text
[chunk -1, 1] [chunk 0, 1] [chunk 1, 1]
[chunk -1, 0] [chunk 0, 0] [chunk 1, 0]
[chunk -1,-1] [chunk 0,-1] [chunk 1,-1]
```

Quando o jogador se move:

1. identifique o chunk atual;
2. gere os chunks próximos;
3. descarregue ou reutilize os distantes;
4. mantenha a seed global e as coordenadas do chunk;
5. preserve alterações feitas pelo jogador.

O último item é importante: regenerar o mesmo chunk não deve apagar uma construção ou item coletado sem uma política de persistência.

## Multiplayer e backend

Em um jogo online, existem duas estratégias:

- o servidor envia o mapa completo;
- o servidor envia seed e parâmetros, e cada cliente gera o mesmo mapa.

Enviar apenas a seed economiza dados, mas exige que todos usem algoritmo, versão, regras e ordem de geração compatíveis.

Mesmo que o cliente gere o mapa, não confie nele para regras importantes. O [[Backend]] deve validar posição, inventário, recompensas e ações relevantes. A seed pode ser transmitida por uma [[Requisição]] em [[JSON]], mas não deve ser tratada como segredo.

## Testes

Teste a camada lógica sem depender da renderização:

- mesma seed produz o mesmo mapa;
- seeds diferentes geram variações aceitáveis;
- início e saída sempre existem;
- início alcança a saída;
- nenhum item obrigatório fica inacessível;
- o mapa respeita tamanho e limites;
- mapas pequenos e grandes não causam erro;
- valores extremos são tratados;
- a geração termina dentro do tempo esperado;
- o renderizador representa a matriz corretamente.

Use [[Testes]] automatizados para guardar seeds que reproduzem bugs. Teste também uma grande quantidade de seeds aleatórias para encontrar casos raros.

## Versionamento e evolução

O resultado de um mapa procedural depende de mais do que a seed:

```text
mapa = função(seed, algoritmo, versão, parâmetros, assets)
```

Alterar qualquer uma dessas entradas pode mudar o mapa. Versione o algoritmo e os parâmetros quando for importante manter compatibilidade.

Com [[Git]], mantenha configurações e scripts versionados. Se os Prefabs, tiles ou regras visuais mudarem, o resultado também pode mudar mesmo com a mesma matriz lógica.

## Boas práticas

1. **Use seed reproduzível.** Isso facilita testes, suporte e compartilhamento.
2. **Separe dados de visual.** Primeiro gere a matriz, depois renderize.
3. **Valide conectividade.** Um mapa aleatório precisa continuar sendo jogável.
4. **Defina limites.** Evite loops sem fim e geração de mapas enormes sem controle.
5. **Tenha um fallback.** Use uma seed segura ou um mapa padrão se a geração falhar.
6. **Use regras claras.** Sorte pura produz resultados imprevisíveis e frequentemente ruins.
7. **Evite gerar tudo como GameObject.** Tilemaps, chunks e pooling podem reduzir custo.
8. **Use Prefabs para conteúdo reutilizável.** Não monte cada objeto complexo manualmente.
9. **Mantenha configurações externas ao código.** ScriptableObjects ajudam a experimentar parâmetros.
10. **Meça o desempenho.** Não presuma que um mapa pequeno continuará pequeno.
11. **Teste muitas seeds.** Casos raros aparecem somente depois de várias gerações.
12. **Versione o algoritmo.** Mudanças podem invalidar mapas salvos ou partidas online.
13. **Não confie no cliente online.** O servidor deve validar regras importantes.
14. **Use [[Design Pattern]] com propósito.** Object Pooling, Strategy e Factory podem ajudar, mas não são obrigatórios.
15. **Documente os parâmetros.** Explique como escala, limites, vizinhos e seed alteram o resultado.

## Resumo

Mapa procedural é um cenário criado automaticamente por regras e algoritmos. Na Unity, um bom fluxo gera uma representação lógica, valida conectividade e só depois cria tiles, GameObjects ou Prefabs. Seeds permitem repetir resultados; ruído, random walk, cellular automata e BSP produzem estilos diferentes. Para um sistema confiável, controle a aleatoriedade, teste muitas seeds, cuide do desempenho e mantenha as regras importantes no servidor em jogos online.

### Veja também

- [[Unity Engine]]
- [[Programação procedural]]
- [[Design Pattern]]
- [[Backend]]
- [[Requisição]]
- [[JSON]]
- [[Git]]
- [[Testes]]

### Referências oficiais

- [Conceitos fundamentais da Unity](https://docs.unity3d.com/Manual/key-concepts.html)
- [Mathf.PerlinNoise](https://docs.unity3d.com/ScriptReference/Mathf.PerlinNoise.html)
- [Introdução a Prefabs](https://docs.unity3d.com/Manual/prefabs-introduction.html)
- [Instanciação de Prefabs em runtime](https://docs.unity3d.com/2019.4/Manual/InstantiatingPrefabs.html)
- [ScriptableObject](https://docs.unity3d.com/Manual/class-ScriptableObject.html)
