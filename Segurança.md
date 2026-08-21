# Segurança

Segurança da informação é o conjunto de práticas usadas para proteger dados, sistemas e pessoas contra acesso indevido, alteração, perda ou interrupção.

Em desenvolvimento, segurança não é uma etapa feita somente no final. Ela deve ser considerada desde o planejamento até a operação do [[Backend]], do [[Frontend]], do [[Banco de dados]] e da infraestrutura.

## Uma analogia

Uma casa segura não depende apenas de uma fechadura. Ela pode ter portão, alarme, iluminação, câmeras, vizinhos atentos e um plano para agir em caso de invasão.

Um sistema funciona de forma parecida: cada camada reduz uma parte do risco. Se uma proteção falhar, outras ainda podem limitar o prejuízo. Essa ideia é chamada de **defesa em profundidade** (*defense in depth*).

## Os três objetivos principais

A segurança costuma ser explicada pela tríade **CIA**:

- **Confidencialidade (*confidentiality*)**: somente pessoas e sistemas autorizados podem ver a informação.
- **Integridade (*integrity*)**: a informação não pode ser alterada sem autorização.
- **Disponibilidade (*availability*)**: o sistema e os dados devem estar acessíveis quando forem necessários.

Por exemplo, criptografar uma senha ajuda na confidencialidade, registrar alterações ajuda na integridade e manter backups e redundância ajuda na disponibilidade.

## Autenticação e autorização

Esses conceitos são diferentes:

- **Autenticação (*authentication*)** responde: “quem é você?”.
- **Autorização (*authorization*)** responde: “o que você pode fazer?”.

Uma pessoa pode estar autenticada e ainda assim não ter permissão para excluir um registro. O [[OAuth 2.0]], o [[OIDC]], o [[PKCE]] e o [[JWT]] podem participar da autenticação e da autorização, mas cada um resolve uma parte diferente do problema.

Uma API deve repetir a verificação no servidor. Esconder um botão no [[Frontend]] não impede que alguém envie diretamente uma [[Requisição]] com a operação proibida.

## Riscos comuns

- **Senha fraca ou reutilizada**: facilita o acesso indevido a várias contas.
- **Injeção (*injection*)**: dados enviados pelo usuário são interpretados como comandos, como em SQL Injection.
- **XSS (*Cross-Site Scripting*)**: um invasor consegue executar [[JavaScript]] malicioso no navegador de outra pessoa.
- **CSRF (*Cross-Site Request Forgery*)**: uma página maliciosa tenta fazer uma ação usando a sessão de outra pessoa.
- **Vazamento de segredo**: uma senha, chave privada ou token aparece no código, no repositório ou nos logs.
- **Controle de acesso incorreto**: a aplicação permite acessar ou alterar um recurso de outra pessoa.
- **Dependência vulnerável**: uma biblioteca ou imagem usada pelo sistema possui uma falha conhecida.
- **Configuração insegura**: portas, serviços, permissões ou mensagens de erro ficam mais expostos do que deveriam.

Conhecer os riscos ajuda a escolher proteções. A segurança não significa eliminar todo risco, mas reduzi-lo e preparar uma resposta caso algo aconteça.

## Boas práticas no desenvolvimento

### Validar e tratar entradas

Considere toda entrada externa como não confiável: formulários, parâmetros de URL, cabeçalhos, arquivos e respostas de outros serviços.

- valide tipo, tamanho, formato e limites no [[Backend]];
- use consultas parametrizadas para evitar SQL Injection no [[PostgreSQL]];
- escape ou sanitize conteúdo que será exibido no navegador;
- não confie apenas na validação feita pelo [[Frontend]].

Exemplo de uma consulta parametrizada:

```sql
SELECT id, nome
FROM usuarios
WHERE email = :email;
```

O valor de `:email` deve ser enviado separadamente pela biblioteca de acesso ao banco. Concatenar texto recebido pela pessoa diretamente no SQL pode permitir que ela altere o comando.

### Proteger senhas e segredos

- nunca salve senhas em texto puro; use um algoritmo de hash próprio para senhas, como Argon2id ou bcrypt;
- mantenha tokens, chaves privadas e senhas fora do código e do [[GitHub]];
- use variáveis de ambiente ou um gerenciador de segredos;
- conceda a cada serviço somente as permissões necessárias, seguindo o princípio do menor privilégio (*least privilege*);
- troque segredos quando houver suspeita de exposição.

Um `.env` local pode ajudar no desenvolvimento, mas não deve ser enviado ao repositório. Também não basta ocultar um segredo no frontend: tudo que é enviado ao navegador pode ser descoberto pelo usuário.

### Proteger a comunicação

Use HTTPS, que combina HTTP com [[TLS]] (*Transport Layer Security*), para impedir que terceiros leiam ou alterem os dados durante o caminho.

```http
Authorization: Bearer <access-token>
```

O token deve ser enviado em um cabeçalho HTTPS, não em uma URL. URLs podem aparecer no histórico do navegador, em proxies e em logs.

### Controlar sessões e tokens

- prefira fluxos modernos, como Authorization Code com [[PKCE]];
- use tokens com duração curta e escopos limitados;
- valide tokens no servidor de recursos;
- diferencie access token de ID token; um ID token de [[OIDC]] não deve ser usado automaticamente para chamar uma API;
- proteja refresh tokens e planeje sua revogação;
- não registre tokens ou credenciais nos logs.

### Atualizar dependências

Mantenha o [[Java]], [[Quarkus]], [[Maven]], bibliotecas do frontend, imagens do [[Docker]] e demais componentes atualizados. Antes de atualizar em produção, leia as mudanças, execute testes e verifique a compatibilidade.

No fluxo de [[CI-CD]], inclua análise de dependências, verificações de código e varredura de imagens. Uma ferramenta ajuda a encontrar problemas, mas não substitui a revisão humana.

## Segurança na operação

- exponha somente as portas necessárias;
- use regras de firewall e grupos de segurança restritivos;
- mantenha o sistema operacional e os serviços atualizados;
- acompanhe logs e alertas sem registrar dados secretos;
- faça backups criptografados, em local separado, e teste a restauração;
- tenha um plano para revogar credenciais, isolar o sistema e comunicar um incidente;
- aplique mudanças de forma rastreável e revise permissões periodicamente.

Em uma [[VPS]], a equipe normalmente configura o firewall, as atualizações, os backups, o acesso SSH e o monitoramento. Na AWS, serviços como EC2, Security Groups, IAM, KMS, Secrets Manager, CloudTrail, GuardDuty e AWS WAF oferecem partes equivalentes, com maior integração e automação. A VPS costuma dar mais simplicidade de custo e controle direto; a AWS oferece serviços gerenciados e escala, mas exige conhecer mais configurações e pode gerar custos por uso.

## Segurança não é apenas esconder informações

Codificar um texto em Base64, por exemplo, não é criptografá-lo. Da mesma forma, um [[JWT]] normalmente pode ser lido por quem o possui; sua assinatura ajuda a detectar alterações, mas não esconde o conteúdo.

A proteção adequada depende da finalidade: criptografia para sigilo, assinatura para verificar integridade e identidade, hash para comparar valores sem guardar o original e controle de acesso para limitar ações.

## Checklist simples

Antes de publicar uma aplicação, pergunte:

- os dados trafegam por HTTPS?
- senhas e segredos estão fora do código, do repositório e dos logs?
- o backend valida entradas e permissões?
- tokens e sessões têm duração e escopos adequados?
- dependências e imagens foram verificadas?
- somente portas e serviços necessários estão expostos?
- os backups foram feitos e a restauração foi testada?
- existe um plano para responder a um incidente?

Segurança é um processo contínuo de reduzir riscos, observar o sistema e melhorar suas proteções.
