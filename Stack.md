# Stack

Em desenvolvimento de sistemas, **stack** é o conjunto de tecnologias (*technology stack*) usadas para criar, executar e publicar uma aplicação.

Podemos imaginar uma aplicação como uma casa. Cada tecnologia tem uma função diferente: uma cuida da aparência, outra das regras, outra guarda os dados e outras ajudam a colocar tudo para funcionar. O conjunto dessas tecnologias é a **stack** do projeto.

## Exemplo da stack deste repositório

Esta stack combina [[Framework|frameworks]], ferramentas, serviços e um ambiente de hospedagem. Cada parte tem uma responsabilidade diferente:

- **[[Backend]] (servidor ou *server-side*):** [[Java 17]], [[Quarkus]] e [[Maven]] (`mvn`). O [[Backend]] recebe pedidos, aplica as regras do sistema e devolve respostas.
- **[[Banco de dados]] (*database*):** [[PostgreSQL]]. É onde as informações ficam guardadas.
- **[[Frontend]] (interface do usuário ou *client-side*):** [[Next.js]] e [[React]]. É a parte que o usuário vê e utiliza.
- **Execução e implantação (*deployment*):** [[Docker Compose]]. Ajuda a iniciar e organizar os serviços da aplicação.
- **Servidor web (*web server*) e proxy:** [[Caddy]]. Recebe as conexões dos usuários e encaminha cada pedido para o serviço correto.
- **Hospedagem (*hosting*):** [[VPS]]. É o computador conectado à internet onde a aplicação pode ficar disponível.

## Por que a stack é importante?

Conhecer a stack ajuda a entender como as partes de um sistema se conectam. Também facilita escolher ferramentas, dividir tarefas e descobrir onde um problema pode estar.

Por exemplo, se uma tela não aparece, o problema pode estar no [[Frontend]]. Se os dados não são salvos, pode estar no [[Backend]] ou no [[PostgreSQL]]. Se a aplicação nem consegue ser acessada, pode haver algo errado no [[Caddy]], no [[Docker Compose]] ou na [[VPS]].

## Stack não é uma tecnologia única

Uma stack não é um programa específico. Ela é uma combinação de ferramentas que trabalham juntas. Projetos diferentes podem usar stacks diferentes, assim como casas podem ser construídas com materiais diferentes.

Também é comum ouvir expressões como **full stack**, que significa conhecer ou trabalhar tanto com o [[Frontend]] quanto com o [[Backend]] e, muitas vezes, também com banco de dados e implantação.

## Boas práticas ao escolher uma stack

- escolher tecnologias que atendam às necessidades do projeto, em vez de seguir apenas uma moda;
- verificar se as tecnologias são compatíveis, evitando problemas de integração;
- manter a stack simples, usando apenas as ferramentas necessárias;
- documentar a função e a versão de cada tecnologia, facilitando a entrada de novas pessoas no projeto;
- manter as ferramentas atualizadas com cuidado, corrigindo problemas de segurança sem quebrar o sistema;
- testar a aplicação depois de trocar uma tecnologia ou atualizar uma versão.

## Resumo

> **Stack é o conjunto de tecnologias que formam a base de uma aplicação.**

Para entender uma stack, pergunte:

1. Qual tecnologia cuida da interface?
2. Qual tecnologia cuida das regras do sistema?
3. Onde os dados são armazenados?
4. Como os serviços são executados?
5. Onde a aplicação fica hospedada?
