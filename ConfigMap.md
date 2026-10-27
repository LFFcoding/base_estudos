# O que é ConfigMap (Config Map)?

Um **ConfigMap** é um objeto Kubernetes que guarda configurações não sigilosas em pares de chave e valor. A aplicação pode recebê-las como variáveis de ambiente, argumentos ou arquivos montados no [[Pod]].

Uma analogia: o ConfigMap é o quadro de instruções da cozinha — informa endereço de serviço, modo de execução ou nome de fila, mas não deve conter a senha do cofre.

Separar configuração da imagem permite usar a mesma imagem em ambientes diferentes. Por exemplo, desenvolvimento e produção podem fornecer endereços de banco distintos sem reconstruir o [[Container]].

## Exemplo

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  API_MODE: "public"
```

Um Deployment pode referenciar uma chave:

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

## Boas práticas

- use ConfigMap somente para valores não confidenciais;
- mantenha manifests versionados e revise alterações por ambiente;
- não presuma que uma variável de ambiente já carregada se atualiza quando o ConfigMap muda; muitas vezes é preciso reiniciar o Pod;
- considere ConfigMaps imutáveis quando a configuração deve mudar apenas com uma nova versão.

## Equivalente na AWS

**AWS Systems Manager Parameter Store** e **AWS AppConfig** podem armazenar ou distribuir configurações, mas não se comportam exatamente como um objeto Kubernetes montado em Pods. Em EKS, é comum combinar serviços AWS com integração e permissões apropriadas.

## Relação com outros assuntos

ConfigMap pertence à área [[Config]]. Para dados sigilosos, use [[Secret]], não ConfigMap.

## Referências

- [ConfigMaps no Kubernetes](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Configuração no Kubernetes](https://kubernetes.io/docs/concepts/configuration/)
