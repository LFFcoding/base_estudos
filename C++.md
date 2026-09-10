# O que é C++?

**C++** (pronuncia-se “C mais mais”) é uma linguagem de programação criada por Bjarne Stroustrup. Ela surgiu como uma evolução da linguagem C e combina controle detalhado do computador com recursos de alto nível, como classes, generics por meio de templates e uma biblioteca padrão completa.

C++ é normalmente compilado para código nativo: o compilador transforma o código-fonte em instruções que o processador consegue executar diretamente. Por isso, a linguagem pode oferecer alto desempenho, mas também exige mais cuidado com memória e tempo de vida dos objetos.

Uma analogia simples: em uma linguagem com mais automações, muitas decisões são tomadas pelo ambiente. Em C++, o programador pode escolher mais detalhes da construção, como o local e o tempo de vida de certos objetos. Isso dá mais controle, mas também aumenta a responsabilidade.

## Para que serve?

C++ é usado em:

- jogos e motores gráficos;
- sistemas operacionais, drivers e aplicações embarcadas;
- programas que precisam de baixa latência;
- aplicações científicas e processamento de dados;
- ferramentas de áudio, vídeo e computação gráfica;
- aplicações desktop;
- [[Biblioteca|bibliotecas]] nativas usadas por outras linguagens;
- partes de sistemas de [[Backend]] que exigem alto desempenho.

Ele é uma linguagem de vários paradigmas: permite [[Programação procedural]], orientação a objetos e técnicas funcionais. Assim, pode ser usado tanto para uma tarefa pequena quanto para sistemas muito grandes.

## Primeiro programa

```cpp
#include <iostream>

int main()
{
    std::cout << "Olá, mundo!\n";
    return 0;
}
```

`#include <iostream>` inclui recursos de entrada e saída. `main` é o ponto inicial do programa, e `std::cout` escreve no terminal.

O arquivo pode ser compilado com um compilador como `g++` ou `clang++`:

```bash
clang++ -std=c++20 -Wall -Wextra -Wpedantic main.cpp -o programa
./programa
```

Nesse exemplo, `-std=c++20` escolhe a versão da linguagem e as opções `-Wall`, `-Wextra` e `-Wpedantic` ativam avisos úteis do compilador. Avisos não devem ser ignorados sem uma razão clara. Os comandos são executados no [[Terminal]].

## Variáveis e tipos

Uma variável guarda um valor e possui um tipo definido:

```cpp
#include <string>

std::string nome = "Ana";
int idade = 30;
double temperatura = 23.5;
bool ativo = true;
const int limite = 10;
```

`const` informa que o valor não deve ser alterado depois da inicialização. Usar `const` sempre que possível deixa as regras do programa mais claras e evita mudanças acidentais.

Para dinheiro, não é uma boa ideia depender diretamente de `float` ou `double`, pois números de ponto flutuante podem apresentar pequenas diferenças de representação. Uma aplicação pode trabalhar com centavos em um tipo inteiro ou usar uma biblioteca decimal adequada, de acordo com a necessidade.

Prefira inicializar as variáveis imediatamente:

```cpp
int quantidade{3};
std::string cidade{"São Paulo"};
```

As chaves ajudam a evitar algumas conversões inesperadas e deixam explícito que a variável já começa com um valor.

## Funções

Uma função agrupa uma tarefa que pode receber dados e devolver um resultado:

```cpp
int somar(int primeiro, int segundo)
{
    return primeiro + segundo;
}

int resultado = somar(2, 3);
```

Funções curtas, com nomes claros e uma responsabilidade bem definida, são mais fáceis de testar e reutilizar. Essa ideia também aparece nos princípios [[SOLID]].

## Referências e ponteiros

Uma referência é outro nome para um objeto que já existe:

```cpp
void adicionarUm(int& numero)
{
    ++numero;
}

int valor{4};
adicionarUm(valor);
```

Como a função recebe `int&`, ela altera o `valor` original. Use referências quando o objeto precisa existir e a função não precisa representar a ausência dele.

Um ponteiro guarda um endereço de memória e pode não apontar para nenhum objeto:

```cpp
int valor{4};
int* ponteiro{&valor};

if (ponteiro != nullptr)
{
    *ponteiro = 5;
}
```

`&valor` obtém o endereço e `*ponteiro` acessa o valor naquele endereço. `nullptr` representa um ponteiro sem objeto associado.

Ponteiros são úteis em algumas situações de baixo nível, mas exigem cuidado. Um ponteiro inválido pode causar falhas difíceis de investigar. Em código moderno, prefira referências quando a ausência não faz sentido e tipos de gerenciamento automático quando houver posse de um objeto.

## Memória e tempo de vida

De forma simplificada:

- a *stack* normalmente guarda variáveis locais e é gerenciada automaticamente ao sair do escopo;
- a *heap* é usada para objetos cuja vida pode ser dinâmica;
- o tempo de vida define quando um objeto é criado e quando seus recursos são liberados.

O erro clássico é liberar uma área de memória duas vezes, usá-la depois de liberá-la ou esquecer de liberá-la. Em C++ moderno, evite fazer `new` e `delete` manualmente em código comum. Prefira **RAII** (*Resource Acquisition Is Initialization*): o objeto adquire o recurso no construtor e o libera automaticamente no destrutor.

```cpp
#include <fstream>

void lerArquivo()
{
    std::ifstream arquivo{"dados.txt"};
    // O arquivo é fechado automaticamente quando sair deste escopo.
}
```

Esse padrão também funciona para mutexes, conexões e outros recursos. O `using` de C# e o gerenciamento de recursos em Java expressam uma preocupação parecida, embora a implementação seja diferente.

## Smart pointers

Quando um objeto precisa ser criado dinamicamente, use ponteiros inteligentes:

```cpp
#include <memory>

auto numero = std::make_unique<int>(42);
```

`std::unique_ptr` representa posse exclusiva: somente um responsável é dono do objeto. Quando o ponteiro sai do escopo, o objeto é liberado.

`std::shared_ptr` permite posse compartilhada, mas deve ser usado com cuidado porque torna o tempo de vida menos óbvio e pode criar ciclos. Sempre que possível, prefira `std::unique_ptr`, referências ou objetos mantidos diretamente.

## Classes e objetos

Uma classe reúne dados e comportamentos. Um objeto é uma instância dessa classe:

```cpp
#include <stdexcept>

class Conta
{
public:
    explicit Conta(double saldoInicial)
        : saldo_{saldoInicial}
    {
        if (saldoInicial < 0)
        {
            throw std::invalid_argument{"Saldo inválido"};
        }
    }

    void depositar(double valor)
    {
        if (valor <= 0)
        {
            throw std::invalid_argument{"O depósito deve ser positivo"};
        }

        saldo_ += valor;
    }

    [[nodiscard]] double saldo() const
    {
        return saldo_;
    }

private:
    double saldo_{};
};

Conta conta{100.0};
conta.depositar(25.0);
```

Nesse exemplo:

- `private` esconde o estado interno;
- `public` expõe as operações permitidas;
- `explicit` evita conversões automáticas indesejadas no construtor;
- `const` no método garante que ele não altere a conta;
- `[[nodiscard]]` avisa quando o chamador ignora um resultado importante;
- o construtor valida o estado inicial.

Esse encapsulamento protege as regras da classe e facilita a evolução do código. O uso de contratos claros também se relaciona a [[Design Pattern]] e a testes bem definidos.

## Classes abstratas e interfaces

C++ não possui uma palavra-chave `interface` como C# ou Java. Uma interface costuma ser modelada por uma classe abstrata com funções virtuais puras:

```cpp
class Notificador
{
public:
    virtual ~Notificador() = default;
    virtual void enviar(const std::string& mensagem) = 0;
};
```

Uma classe derivada precisa implementar `enviar`. O destrutor virtual é importante quando um objeto derivado pode ser destruído por meio de um ponteiro para a classe base.

Use herança quando existir uma relação realmente adequada. Muitas vezes, composição — uma classe contendo outra — produz um design mais simples e flexível.

## Biblioteca padrão e STL

A biblioteca padrão do C++ oferece estruturas e algoritmos prontos, conhecidos em conjunto como STL (*Standard Template Library*):

```cpp
#include <string>
#include <unordered_map>
#include <vector>

std::vector<std::string> nomes{"Ana", "Bruno", "Carla"};
std::unordered_map<int, std::string> usuarios{
    {1, "Ana"},
    {2, "Bruno"}
};
```

Algumas estruturas comuns são:

- `std::vector`: sequência com acesso rápido por posição;
- `std::array`: sequência de tamanho fixo;
- `std::map`: associa chaves a valores mantendo as chaves ordenadas;
- `std::unordered_map`: associa chaves a valores sem ordenar, geralmente com acesso rápido;
- `std::optional`: representa um valor que pode não existir;
- `std::string`: representa texto.

Prefira estruturas da biblioteca padrão a criar versões próprias sem necessidade. Elas foram amplamente utilizadas e têm comportamento documentado.

## Algoritmos e ordenação

O cabeçalho `<algorithm>` fornece operações reutilizáveis:

```cpp
#include <algorithm>
#include <vector>

std::vector<int> valores{4, 1, 3, 2};
std::sort(valores.begin(), valores.end());
```

Depois da chamada, os valores estão em ordem crescente. A biblioteca também possui funções como `find`, `count`, `copy` e `transform`. Usar algoritmos prontos costuma deixar a intenção mais clara do que reimplementar loops. O assunto se conecta a [[Sorting]].

## Templates

Templates permitem escrever uma operação que funciona com vários tipos:

```cpp
template <typename T>
T maior(T primeiro, T segundo)
{
    return primeiro > segundo ? primeiro : segundo;
}

int maiorInteiro = maior(3, 7);
double maiorDecimal = maior(2.5, 1.8);
```

O compilador cria a versão necessária para cada tipo usado. Isso oferece reutilização e desempenho, mas mensagens de erro de templates podem ser longas. Mantenha templates pequenos e documente as exigências dos tipos.

## Tratamento de erros

C++ pode usar exceções para informar falhas que impedem a continuação normal:

```cpp
try
{
    Conta conta{100.0};
    conta.depositar(-1.0);
}
catch (const std::invalid_argument& erro)
{
    std::cerr << erro.what() << '\n';
}
```

Capture exceções por referência constante quando não precisar modificá-las. Não use `catch (...)` apenas para esconder erros. Em componentes que precisam de desempenho previsível ou não podem usar exceções, um tipo de resultado ou código de erro pode ser mais adequado, desde que a regra seja consistente.

## Concorrência

C++ oferece recursos para executar tarefas em paralelo, como `std::thread`, `std::future`, mutexes e, em versões modernas, `std::jthread`.

Concorrência pode reduzir o tempo de uma tarefa, mas também cria riscos de condição de corrida, deadlock e acesso simultâneo a dados inválidos. Proteja o estado compartilhado, mantenha regiões críticas pequenas e prefira abstrações que expressem claramente quem é responsável por cada recurso.

Não crie threads sem necessidade. Uma fila de tarefas ou uma biblioteca especializada pode ser mais adequada para um sistema real.

## C++ em APIs e sistemas

C++ pode criar APIs e serviços de [[Backend]], especialmente quando latência e consumo de recursos são prioridades. Uma aplicação pode receber uma [[Requisição]], validar os dados, acessar um banco e devolver [[JSON]]. Para isso, costuma usar bibliotecas externas de HTTP, JSON e acesso a banco; a escolha depende do sistema e das restrições do projeto.

Mesmo com alto desempenho, as boas práticas de segurança continuam necessárias:

- valide tamanhos e formatos antes de copiar dados;
- evite acessos fora dos limites de vetores e buffers;
- use ferramentas de análise e sanitizers para encontrar erros de memória;
- não confie em dados recebidos da rede;
- não coloque credenciais no código;
- registre erros sem expor informações sensíveis;
- aplique autenticação e autorização conforme a operação;
- siga as orientações do arquivo [[Segurança]].

## Relação com C# e Java

C++ tem classes, templates, exceções e uma biblioteca padrão, assim como C# e [[Java]] têm classes, generics e bibliotecas. As três linguagens podem ser usadas em sistemas grandes, mas o gerenciamento de recursos é diferente:

- C++ oferece controle direto sobre o tempo de vida dos objetos e usa RAII; não possui um coletor de lixo obrigatório;
- C# normalmente roda sobre o .NET e usa coleta automática de lixo, além de `using` para recursos que precisam ser fechados;
- Java roda na JVM e também usa coleta automática de lixo;
- C++ costuma gerar código nativo e exige mais atenção a ponteiros, referências e limites de memória;
- C# e Java tendem a oferecer mais automações do ambiente de execução.

O código C++ não é automaticamente código C# ou Java. Aprender os conceitos de uma linguagem ajuda, mas cada ecossistema tem APIs, compiladores e convenções próprias.

## C++ e Unity

A [[Unity Engine]] usa C# como linguagem principal para os scripts de gameplay. C++ pode aparecer em plugins nativos, bibliotecas de baixo nível ou ferramentas externas, mas o código comum de comportamento dos GameObjects é escrito em C#.

Por isso, quem estuda desenvolvimento de jogos pode encontrar as duas linguagens: C# para scripts na Unity e C++ em outros motores, bibliotecas gráficas ou componentes nativos. O assunto também se relaciona a [[Mapa procedural]], que pode ser implementado em qualquer uma das duas linguagens, conforme o motor e a plataforma escolhidos.

## Compilação e organização

Projetos maiores normalmente separam declarações em arquivos de cabeçalho (`.h` ou `.hpp`) e implementações em arquivos `.cpp`. Um arquivo de cabeçalho pode ser incluído em vários pontos, por isso é importante evitar inclusões repetidas com `#pragma once` ou *include guards*.

Em vez de compilar cada arquivo manualmente, projetos reais costumam usar ferramentas de configuração e build, como CMake, e depois compiladores como GCC, Clang ou MSVC. A configuração deve declarar as versões, avisos, dependências e opções de compilação de forma reproduzível.

Separar o código em componentes pequenos facilita compilação, manutenção e [[Testes]].

## Boas práticas resumidas

- prefira RAII e tipos da biblioteca padrão;
- evite `new` e `delete` manuais em código comum;
- use `std::unique_ptr` quando existir posse exclusiva;
- use `const` sempre que algo não precisar ser alterado;
- inicialize as variáveis imediatamente;
- prefira referências quando o objeto obrigatoriamente existir;
- valide tamanhos, índices e dados vindos de fora;
- compile com avisos fortes e use sanitizers durante o desenvolvimento;
- escreva [[Testes]] para regras importantes e casos de limite;
- use ferramentas de [[Linting]] e formatação automática;
- mantenha funções e classes com responsabilidades claras;
- evite otimização prematura: meça antes de mudar o código;
- documente decisões de baixo nível e regras de tempo de vida;
- versione o código com [[Git]].

## Em uma frase

**C++ é uma linguagem compilada e de alto desempenho que oferece grande controle sobre o computador, mas exige cuidado especial com memória, tempo de vida dos objetos e concorrência.**

## Fontes

- [C++ — ISO C++](https://isocpp.org/)
- [C++ language reference — Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/cpp-language-reference?view=msvc-170)
- [C++ standard library reference — cppreference](https://en.cppreference.com/w/cpp)
