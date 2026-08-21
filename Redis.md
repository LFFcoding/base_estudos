# Redis

**Redis** é um armazenamento de dados em memória (*in-memory data store*) que trabalha com chaves, valores e estruturas de dados. Ele é muito usado para responder rapidamente a informações temporárias ou acessadas com frequência.

Redis não precisa ser usado como substituto do [[PostgreSQL]]. Em muitas aplicações, ele fica ao lado do banco principal para funcionar como [[Cache|cache]], armazenamento de sessão, [[Filas|fila]] ou mecanismo de comunicação entre serviços.

Redis é frequentemente classificado como uma solução [[NoSQL]] de chave-valor e estruturas de dados, embora seu uso em memória e suas características de persistência precisem ser analisados separadamente.

## Uma analogia

Imagine que o PostgreSQL seja um grande depósito organizado e o Redis seja uma pequena prateleira ao lado do balcão.

- o depósito guarda o estoque completo e durável;
- a prateleira deixa os itens mais procurados à mão;
- se um item não estiver na prateleira, alguém busca no depósito e o coloca ali;
- se a prateleira for esvaziada, os itens podem ser buscados novamente no depósito.

Essa prateleira é rápida, mas não deve ser tratada automaticamente como a única cópia das informações importantes.

## Por que o Redis é rápido?

O Redis mantém grande parte dos dados na memória RAM, que é muito mais rápida do que acessar um disco a cada operação. Ele também possui comandos simples e estruturas de dados prontas.

A memória é limitada e costuma ser mais cara que o armazenamento em disco. Por isso, o Redis precisa ser dimensionado e configurado com cuidado, principalmente quando guarda dados que não podem ser perdidos.

## Estruturas de dados

Redis não trabalha apenas com texto simples. Algumas estruturas comuns são:

- **String**: texto, número ou conteúdo serializado, como um objeto em [[JSON]];
- **Hash**: conjunto de campos e valores dentro de uma chave, parecido com um pequeno objeto;
- **List**: sequência ordenada de valores, útil para algumas filas;
- **Set**: conjunto de valores sem repetição;
- **Sorted set**: conjunto de valores ordenados por uma pontuação;
- **Stream**: sequência de eventos que pode ser consumida por grupos de consumidores.

A estrutura deve ser escolhida de acordo com a operação necessária. Usar uma lista quando o sistema precisa consultar por pontuação, por exemplo, pode deixar o código mais complicado e menos eficiente.

## Usos comuns

### [[Cache]]

O **cache** guarda temporariamente o resultado de uma operação cara ou repetida.

Um fluxo comum é:

1. o [[Backend]] recebe uma [[Requisição]];
2. procura o resultado no Redis;
3. se encontrar, devolve o valor rapidamente (*cache hit*);
4. se não encontrar (*cache miss*), consulta o [[PostgreSQL]];
5. salva o resultado no Redis com um tempo de expiração;
6. devolve a resposta.

O tempo de expiração é chamado de **TTL** (*time to live*). Ele evita que um dado antigo fique no cache para sempre.

### Sessões

Uma aplicação pode guardar sessões de login no Redis para que vários servidores consigam consultar a mesma sessão. Isso é útil quando o tráfego é distribuído entre várias instâncias do backend.

Mesmo assim, sessões contêm dados sensíveis. Use expiração, proteção de acesso e as práticas de [[Segurança]].

### [[Filas]] e processamento assíncrono

Uma fila permite receber um trabalho agora e executá-lo depois. Por exemplo, o backend pode colocar um pedido para enviar e-mail em uma fila, enquanto um worker processa a tarefa em segundo plano.

Listas e Streams podem ajudar nesse cenário, mas é importante escolher a estratégia de confirmação, reprocessamento e tratamento de falhas. Uma fila simples não substitui automaticamente um sistema de [[Protocolos de mensageria|mensageria]] completo.

### Pub/Sub

No modelo **Publish/Subscribe** (*pub/sub*), um publicador envia uma mensagem para um canal e os assinantes recebem o que está sendo publicado.

Esse recurso é útil para avisos em tempo real, como atualizar telas conectadas. Um assinante que estiver desconectado pode perder a mensagem; quando a entrega precisa ser recuperada depois, uma estrutura persistente, como Streams, pode ser mais adequada.

### Contadores e limites

Redis pode incrementar contadores de forma atômica. Isso ajuda em contagem de acessos, limites de tentativas e **rate limiting** (limitação de frequência).

Um exemplo conceitual é permitir somente uma quantidade de tentativas por minuto para cada usuário. A chave pode ser criada com expiração de 60 segundos e incrementada a cada nova tentativa.

## Comandos básicos

O Redis pode ser acessado pelo cliente de terminal, chamado `redis-cli`:

```text
SET usuario:42 '{"id":42,"nome":"Ana"}' EX 300
GET usuario:42
TTL usuario:42
DEL usuario:42
```

- `SET` salva um valor;
- `EX 300` faz a chave expirar em 300 segundos;
- `GET` lê o valor;
- `TTL` informa quanto tempo falta para expirar;
- `DEL` remove a chave.

Um exemplo usando um hash é:

```text
HSET usuario:42 nome "Ana" email "ana@example.com"
EXPIRE usuario:42 300
HGETALL usuario:42
```

O primeiro comando salva campos separados dentro da chave. O segundo aplica a expiração à chave inteira.

## Configuração local com Docker Compose

Para estudar localmente, o Redis pode ser executado em um [[Container]] definido no [[Docker Compose]]:

```yaml
services:
  redis:
    image: redis:7-alpine
    command: ["redis-server", "--appendonly", "yes"]
    ports:
      - "127.0.0.1:6379:6379"
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

- `appendonly yes` ativa o modo AOF (*Append Only File*) como uma opção de persistência;
- o volume mantém os arquivos de dados fora do ciclo de vida do container;
- `127.0.0.1` limita o acesso publicado à própria máquina durante o estudo local;
- em um projeto real, fixe uma versão aprovada da imagem em vez de usar uma tag que muda sem controle.

Esse exemplo não deve ser copiado diretamente para produção. O ambiente de produção precisa de autenticação, rede privada, backups, monitoramento, política de memória e uma estratégia de alta disponibilidade adequados.

## Persistência

Embora seja orientado à memória, o Redis pode gravar dados no disco. As opções mais conhecidas são:

- **RDB**: cria snapshots do conjunto de dados em determinados momentos;
- **AOF** (*Append Only File*): registra as operações de escrita para reconstruir os dados;
- **RDB + AOF**: combina as duas estratégias;
- **sem persistência**: adequado para alguns caches que podem ser reconstruídos.

Persistência não significa que o Redis deve substituir o banco principal. Defina quanto de perda de dados é aceitável, faça backups e teste a restauração. Para um cache, perder os dados pode ser aceitável; para pedidos ou pagamentos, normalmente não.

## Redis e PostgreSQL

| Característica | Redis | PostgreSQL |
| --- | --- | --- |
| Modelo | chaves e estruturas de dados em memória | banco relacional com tabelas e SQL |
| Uso comum | cache, sessão, contadores e filas | dados principais e relacionamentos |
| Velocidade típica | muito alta para operações simples em memória | adequada para consultas complexas e dados duráveis |
| Consultas | comandos e estruturas específicas | SQL, filtros, junções e agregações |
| Persistência | configurável | parte central do uso do banco |

Uma aplicação pode usar os dois: o PostgreSQL como fonte principal da verdade e o Redis para acelerar ou coordenar operações.

## Redis na arquitetura

Uma arquitetura simples pode ficar assim:

1. o [[Frontend]] envia dados para o [[Backend]];
2. o backend valida a requisição e consulta o Redis;
3. o Redis responde rapidamente quando possui o valor;
4. em um *cache miss*, o backend consulta o PostgreSQL;
5. o backend pode armazenar uma cópia temporária no Redis;
6. a resposta volta para o frontend em [[JSON]].

Redis não substitui o backend, as regras de negócio ou a autorização. Ele é um componente que participa de alguns fluxos.

## Segurança

Nunca deixe uma instância Redis de produção aberta para toda a internet. Boas práticas incluem:

- manter o Redis em uma rede privada;
- permitir acesso somente dos serviços necessários;
- usar autenticação e ACLs (*Access Control Lists*) com permissões mínimas;
- usar TLS quando os dados trafegarem por uma rede não confiável;
- proteger as chaves e credenciais em um gerenciador de segredos;
- não salvar senhas, tokens ou dados pessoais sem necessidade;
- definir TTL para sessões, códigos temporários e caches;
- limitar memória e escolher uma política de remoção (*eviction*) adequada;
- acompanhar conexões, erros, memória, latência e chaves que não expiram;
- não usar comandos administrativos ou acesso público em ambientes de produção.

Redis é rápido, mas velocidade não é uma proteção. Uma instância exposta ou sem autenticação pode permitir leitura, alteração ou exclusão de dados.

## Redis em uma VPS e na AWS

Em uma [[VPS]], a equipe pode instalar o Redis diretamente ou executá-lo em um [[Container]] usando [[Docker]], configurando rede, atualização, autenticação, persistência, backups, monitoramento e recuperação.

Na AWS, o [Amazon ElastiCache for Valkey and Redis OSS](https://aws.amazon.com/elasticache/redis/) é a opção gerenciada mais próxima para um cache compatível com APIs Redis. A AWS assume mais partes da operação e oferece recursos de segurança, escalabilidade e alta disponibilidade, mas o serviço gera cobrança e aumenta a dependência da plataforma.

O [Amazon MemoryDB](https://docs.aws.amazon.com/memorydb/) também é um banco em memória gerenciado e compatível com Valkey e Redis OSS, mas é voltado a cenários que precisam de maior durabilidade e disponibilidade para os dados em memória. Ele não deve ser escolhido automaticamente: para um cache reconstruível, ElastiCache costuma representar melhor o objetivo; para dados em memória que precisam ser mais duráveis, MemoryDB pode ser avaliado.

As opções gerenciadas reduzem o trabalho operacional, mas não eliminam a necessidade de escolher TTL, políticas de remoção, permissões, formato das chaves, custo e estratégia de recuperação.

## Boas práticas

- use nomes de chave previsíveis e com espaços de nomes, como `app:usuario:42`;
- defina TTL para dados temporários;
- invalide ou atualize o cache quando a fonte principal mudar;
- prepare-se para *cache stampede*, quando muitas requisições tentam reconstruir a mesma chave ao mesmo tempo;
- trate o Redis como indisponível e decida se a aplicação deve usar um fallback;
- evite armazenar objetos enormes ou listas sem limite;
- prefira comandos atômicos para contadores e limites;
- use locks distribuídos somente entendendo seus riscos de expiração e concorrência;
- monitore memória, latência, acertos, falhas e remoções do cache;
- teste reinício, perda do cache, restauração e comportamento de fallback;
- não dependa de dados reais em uma instância local compartilhada.

**Redis é um armazenamento rápido e versátil, especialmente útil para dados temporários, comunicação e operações que precisam de baixa latência.** Seu uso correto depende de entender memória, expiração, persistência, segurança e o papel do PostgreSQL como fonte principal dos dados.
