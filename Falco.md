# O que é Falco?

**Falco** é uma ferramenta de segurança em runtime (*durante a execução*) que observa eventos do sistema e compara esses eventos com regras. Pode detectar situações como um shell iniciado dentro de um container, alterações incomuns em arquivos ou comportamento inesperado de processos.

Uma analogia: Trivy inspeciona o prédio antes da abertura; Falco é um alarme que acompanha o que acontece enquanto o prédio está funcionando.

Em Kubernetes, Falco pode enriquecer eventos com informações de Pods, namespaces e containers. Por padrão, detecção e alerta não significam automaticamente bloquear a ação; respostas adicionais dependem de integrações e precisam ser desenhadas com cuidado.

## Exemplo de uso

Uma equipe pode criar uma regra que gere um alerta se um processo de shell for iniciado em um container de produção que normalmente não precisa de terminal interativo. A regra deve considerar exceções legítimas para evitar excesso de alertas.

## Boas práticas

- comece com regras conhecidas e ajuste-as ao comportamento normal da aplicação;
- teste regras e integrações antes de automatizar bloqueios;
- proteja alertas e metadados, que podem revelar detalhes do ambiente;
- combine detecção em runtime com menor privilégio, isolamento e atualização de imagens.

## Equivalente na AWS

**Amazon GuardDuty Runtime Monitoring** observa eventos de sistema, processos, arquivos e rede em workloads compatíveis e gera findings de ameaças. É uma alternativa gerenciada para parte da detecção de runtime, mas não executa as mesmas regras personalizadas do Falco; a cobertura varia conforme o tipo de nó e ambiente, incluindo restrições para workloads EKS em Fargate.

## Relação com outros assuntos

Falco pertence a [[Security]]. Complementa [[Trivy]], que procura problemas antes da execução, e [[RBAC]], que controla chamadas à API Kubernetes.

## Referências

- [Documentação oficial do Falco](https://falco.org/docs/)
- [GuardDuty Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html)
