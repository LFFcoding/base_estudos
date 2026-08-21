# Cache

**Cache** é um armazenamento temporário de dados que podem ser obtidos novamente. Ele existe para evitar trabalho repetido e diminuir o tempo de resposta.

Um cache não costuma ser a fonte principal da verdade. Se ele desaparecer, a aplicação deve conseguir reconstruir os dados a partir de outra fonte, como o [[PostgreSQL]].

## Uma analogia

Imagine uma mesa de estudos com os livros usados hoje:

- a biblioteca completa é o banco principal;
- a mesa guarda apenas o material mais procurado;
- buscar algo na mesa é rápido;
- se o material não estiver ali, você volta à biblioteca;
- depois, pode deixar uma cópia na mesa para a próxima consulta.

A mesa não precisa guardar todos os livros. Ela precisa guardar os itens certos pelo tempo certo.

## Cache hit e cache miss

Quando uma aplicação procura um valor:

- **cache hit**: o valor está no cache e pode ser devolvido rapidamente;
- **cache miss**: o valor não está no cache, então a aplicação busca na fonte principal.

Um fluxo comum, chamado **cache-aside**, é:

1. o [[Backend]] recebe uma [[Requisição]];
2. procura a chave no cache;
3. se encontrar, devolve o valor;
4. se não encontrar, consulta o banco ou outro serviço;
5. salva uma cópia com expiração;
6. devolve a resposta.

O cache-aside é simples, mas a aplicação precisa decidir quando atualizar ou remover a cópia.

## O que pode ser armazenado?

Um cache pode guardar:

- respostas de consultas frequentes;
- configurações que mudam pouco;
- sessões temporárias;
- resultados de cálculos caros;
- páginas, imagens e arquivos em uma CDN;
- tokens ou códigos temporários, quando houver proteção adequada;
- contadores e dados de limitação de frequência.

O valor pode ser um texto, número, objeto serializado em [[JSON]] ou outra representação adequada.

## TTL e expiração

**TTL** significa *time to live* (tempo de vida). Ele define por quanto tempo uma entrada pode permanecer no cache.

```text
SET produto:10 '{"id":10,"nome":"Caderno"}' EX 300
```

Esse comando do [[Redis]] salva o produto por 300 segundos. Depois desse período, o valor expira e a próxima consulta precisa buscá-lo novamente.

O TTL deve considerar a frequência de alteração do dado. Um preço pode precisar de poucos segundos; uma configuração pública pode durar mais. Um TTL muito longo aumenta o risco de dados antigos. Um TTL muito curto reduz o benefício do cache.

## Invalidação

**Invalidação** é retirar ou atualizar uma entrada quando a fonte principal muda.

Exemplo:

1. o backend altera o nome do produto no PostgreSQL;
2. remove `produto:10` do cache;
3. a próxima leitura busca o dado novo;
4. o novo resultado é colocado no cache.

“Há somente duas coisas difíceis na Ciência da Computação: invalidação de cache e nomear coisas” é uma frase famosa porque dados antigos são fáceis de criar e difíceis de controlar.

Estratégias comuns:

- **cache-aside**: aplicação lê e atualiza o cache;
- **write-through**: a escrita atualiza fonte principal e cache;
- **write-behind**: o cache recebe primeiro e grava depois, com risco maior se houver falha;
- **refresh-ahead**: valores populares são renovados antes de expirar.

Escolha a estratégia conforme a importância da consistência, da disponibilidade e da velocidade.

## Tipos de cache

- **cache no navegador**: guarda recursos e respostas no cliente;
- **CDN**: mantém arquivos ou respostas em pontos próximos das pessoas;
- **cache da aplicação**: fica entre o backend e a fonte de dados;
- **cache de banco**: pode manter páginas ou resultados em memória;
- **cache distribuído**: é compartilhado por várias instâncias do backend.

O local do cache muda o que ele consegue enxergar, a forma de invalidar e o risco de servir um dado incorreto.

## Problemas comuns

### Dados antigos (*stale data*)

O cache pode devolver um valor que já mudou na fonte principal. Defina TTL e invalidação conforme a necessidade de consistência.

### Cache stampede

Muitas entradas populares expiram ao mesmo tempo. Várias requisições percebem o *miss* e consultam a fonte simultaneamente, causando sobrecarga.

Use jitter no TTL, renovação antecipada, bloqueios controlados ou coalescência de requisições quando necessário.

### Cache penetration

Muitas consultas procuram dados que não existem. A aplicação pode repetir a mesma consulta cara sem nunca encontrar um valor.

Em alguns casos, um resultado vazio com TTL curto ou um filtro de entrada ajuda a reduzir o problema.

### Cache avalanche

Uma falha ou expiração em massa faz muitas consultas voltarem ao banco de uma vez. Monitore o cache e mantenha um comportamento de fallback.

## Redis como cache

O [[Redis]] é uma opção popular para cache distribuído porque trabalha em memória, possui expiração por chave e oferece estruturas de dados úteis.

Ele não deve ser colocado na internet sem proteção. Use rede privada, autenticação, ACLs, TLS, limite de memória e uma política de remoção adequada. Para informações que não podem ser perdidas, não dependa apenas de um cache.

## Cache na AWS

Na AWS, o [Amazon ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/dg/WhatIs.corecomponents.html) oferece cache gerenciado com engines como Valkey, Redis OSS e Memcached. A AWS assume parte da administração de nós, capacidade e integração de rede, mas o serviço gera custo e exige configuração de permissões, sub-redes, segurança, expiração e estratégia de recuperação.

CloudFront também pode funcionar como cache de conteúdo na borda para arquivos e respostas HTTP. Ele atende a um problema diferente do Redis: fica próximo das pessoas visitantes e é voltado principalmente à distribuição de conteúdo web.

Em uma [[VPS]], a equipe pode executar [[Redis]] com [[Docker Compose]], mas precisa cuidar de memória, atualização, persistência, backups, monitoramento e disponibilidade.

## Segurança

- não guarde segredos no cache sem necessidade;
- proteja sessões, tokens e dados pessoais com controle de acesso e expiração;
- não confie em um valor do cache sem validar se ele ainda pode ser usado;
- não permita que uma pessoa escolha livremente qualquer chave para ler;
- evite incluir dados sensíveis em nomes de chave ou mensagens de erro;
- use TLS quando o tráfego atravessar uma rede não confiável;
- configure limites de memória e monitore remoções inesperadas;
- trate uma falha do cache sem expor dados ou interromper operações críticas.

Cache é uma otimização, não uma autorização. Esconder ou encontrar um dado no cache não decide se uma pessoa pode acessá-lo; essa regra continua no [[Backend]] e nas práticas de [[Segurança]].

## Boas práticas

- escolha chaves claras e versionáveis, como `app:v1:produto:10`;
- defina TTL para toda informação temporária;
- invalide o cache quando a fonte principal mudar;
- armazene somente o que reduz trabalho ou tempo de forma significativa;
- limite tamanho dos valores e das listas;
- meça taxa de hit, miss, latência, memória e erros;
- tenha fallback para a fonte principal quando o cache estiver indisponível;
- evite usar o cache como banco principal sem entender persistência e recuperação;
- teste dados antigos, expiração, reinício e picos de acesso.

**Cache é uma cópia temporária usada para acelerar leituras ou evitar trabalho repetido.** Seu valor depende de expiração, invalidação, consistência e de uma fonte principal confiável.
