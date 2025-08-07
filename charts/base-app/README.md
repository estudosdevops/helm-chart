# Helm Chart: base-app

Um Helm Chart genérico e altamente configurável, projetado para servir como a base para o deploy de qualquer aplicação no Kubernetes. Ele suporta múltiplos tipos de carga de trabalho, incluindo `Deployment`, `StatefulSet` e `Rollout` do Argo Rollouts.

## 🎯 Propósito

O objetivo deste chart é padronizar a forma como as aplicações são implantadas, garantindo que todas sigam as mesmas convenções de nomenclatura, labels, probes de saúde e configurações de segurança. Ao usá-lo como uma dependência, você acelera o processo de criação de novos serviços e simplifica a manutenção.

## 🚀 Como Usar

Este chart foi projetado para ser usado como uma **dependência** em outros charts.

1. **Adicione a dependência** ao `Chart.yaml` do seu serviço:

    ```yaml
    # no Chart.yaml do seu serviço
    apiVersion: v2
    name: meu-servico-wrapper
    version: 0.1.0

    dependencies:
      - name: base-app
        version: "1.0.0" # A versão do seu chart base-app
        repository: "https://url-do-seu-repositorio-de-charts"
    ```

2. **Configure os valores** no `values.yaml` do seu serviço, aninhando as configurações sob uma chave com o nome da dependência (`base-app`):

    ```yaml
    # no values.yaml do seu serviço
    base-app:
      nameOverride: "meu-servico"
      pods:
        replicas: 3
        image:
          name: "meu-registro/meu-servico"
          # A tag será injetada pelo opsmaster
          tag: "placeholder"
      service:
        enabled: true
    ```

---

## 🔧 Parâmetros de Configuração

A seguir, uma lista dos principais parâmetros que você pode configurar no seu `values.yaml`.

### Configurações Gerais

| Parâmetro      | Descrição                                                                    | Padrão |
| :------------- | :--------------------------------------------------------------------------- | :----- |
| `nameOverride` | Sobrescreve o nome de todos os recursos criados, ignorando o nome do release. | `""`   |

### Configurações dos Pods (`pods`)

| Parâmetro              | Descrição                                                                                             | Padrão                    |
| :--------------------- | :---------------------------------------------------------------------------------------------------- | :------------------------ |
| `controller`           | O tipo de controlador de carga de trabalho a ser usado: `deployment`, `statefulset` ou `daemonset`. É ignorado se `rollout.enabled` for `true`. | `deployment`              |
| `image.name`           | O nome/repositório da imagem do contêiner.                                                               | `""`                      |
| `image.tag`            | A tag da imagem do contêiner.                                                                            | `.Chart.AppVersion`       |
| `image.pullPolicy`     | A política de pull da imagem.                                                                            | `IfNotPresent`            |
| `image.pullSecrets`    | Lista de `imagePullSecrets` existentes para usar ao puxar imagens de um registro privado.                | `[]`                      |
| `imageCredentials`     | Cria automaticamente um segredo com credenciais para um registro privado. Não usar junto com `pullSecrets`. | `{}`                      |
| `replicas`             | O número de réplicas desejadas para o pod.                                                               | `1`                       |
| `revisionHistoryLimit` | O número de `ReplicaSets` antigos a serem mantidos.                                                      | `3`                       |
| `strategy`             | A estratégia de atualização para Deployments/StatefulSets.                                               | `{ type: RollingUpdate }` |
| `livenessProbe`        | Configuração do liveness probe.                                                                          | `{}`                      |
| `readinessProbe`       | Configuração do readiness probe.                                                                         | `{}`                      |
| `resources`            | Limites e requisições de CPU/Memória.                                                                    | `{}`                      |
| `env`                  | Lista de variáveis de ambiente a serem injetadas no contêiner.                                           | `[]`                      |
| `affinity`             | Configurações de afinidade de pods.                                                                      | `{}`                      |
| `tolerations`          | Configurações de tolerations para agendamento em nós com taints.                                         | `[]`                      |

### Configurações de Rollout (`rollout`)

| Parâmetro  | Descrição                                                                                             | Padrão |
| :--------- | :---------------------------------------------------------------------------------------------------- | :----- |
| `enabled`  | Se `true`, o chart irá gerar um objeto `Rollout` em vez de um `Deployment` ou `StatefulSet`.             | `false`  |
| `strategy` | A estratégia de deploy do Argo Rollouts a ser usada (`canary` ou `blueGreen`).                          | `{}`     |

### Configurações de Serviço (`service`)

| Parâmetro | Descrição                                                          | Padrão |
| :-------- | :----------------------------------------------------------------- | :----- |
| `enabled` | Se `true`, cria um `Service` do tipo ClusterIP para a aplicação.     | `false`  |
| `ports`   | Lista de portas a serem expostas pelo serviço.                     | `[]`     |

### Configurações de Serviço Headless (`serviceHeadless`)

Um "Headless Service" é útil para casos de uso stateful (como bancos de dados) ou para descoberta de serviço direta aos pods, sem um load balancer.

| Parâmetro | Descrição                                                  | Padrão |
| :-------- | :--------------------------------------------------------- | :----- |
| `enabled` | Se `true`, cria um `Service` Headless adicional.           | `false`  |
| `ports`   | Lista de portas a serem expostas pelo serviço headless.    | `[]`     |

### Configurações de Ingress (`ingress`)

| Parâmetro     | Descrição                                                 | Padrão |
| :------------ | :-------------------------------------------------------- | :----- |
| `enabled`     | Se `true`, cria um recurso `Ingress` para expor o serviço. | `false`  |
| `hosts`       | Lista de hosts para as regras do Ingress.                 | `[]`     |
| `annotations` | Anotações a serem adicionadas ao Ingress.                 | `{}`     |

### Configurações de Autoscaling

| Parâmetro             | Descrição                                                                                             | Padrão |
| :-------------------- | :---------------------------------------------------------------------------------------------------- | :----- |
| `autoscaling.enabled` | Se `true`, cria um `HorizontalPodAutoscaler` nativo do Kubernetes. É ignorado se `scaling.enabled` for `true`. | `false`  |
| `scaling.enabled`     | Se `true`, cria um `ScaledObject` do KEDA para autoscaling avançado.                                     | `false`  |

### Configurações de External Secrets (`externalSecrets`)

| Parâmetro                 | Descrição                                                                           | Padrão      |
| :------------------------ | :---------------------------------------------------------------------------------- | :---------- |
| `enabled`                 | Se `true`, cria um recurso `ExternalSecret` para sincronizar secrets externos.      | `false`     |
| `refreshInterval`         | Intervalo de sincronização do secret.                                               | `"60s"`     |
| `secretStore.name`        | Nome do `SecretStore` ou `ClusterSecretStore` a ser usado.                         | `""`        |
| `secretStore.kind`        | Tipo do secret store (`SecretStore` ou `ClusterSecretStore`).                      | `"SecretStore"` |
| `dataFrom`                | Lista de configurações para importar todas as chaves de um caminho específico.      | `[]`        |
| `data`                    | Lista de mapeamentos específicos de chaves do secret externo para o secret local.   | `[]`        |

## 📚 Exemplos de Uso

### Exemplo 1: Usando dataFrom (Importar todas as chaves)

```yaml
base-app:
  nameOverride: "minha-app"
  
  externalSecrets:
    enabled: true
    secretStore:
      name: "vault-secret-store"
      kind: "SecretStore"
    dataFrom:
      - key: "secret/minha-app"
```

### Exemplo 2: Usando data (Mapeamento específico)

```yaml
base-app:
  nameOverride: "minha-app"
  
  externalSecrets:
    enabled: true
    secretStore:
      name: "vault-secret-store"
      kind: "ClusterSecretStore"
    data:
      - secretKey: "database-password"
        remoteRef:
          key: "secret/database"
          property: "password"
      - secretKey: "api-token"
        remoteRef:
          key: "secret/api"
          property: "token"
```

### Exemplo 3: Uso misto (dataFrom + data)

```yaml
base-app:
  nameOverride: "minha-app"
  
  externalSecrets:
    enabled: true
    secretStore:
      name: "vault-secret-store"
    # Importa todas as chaves de configuração
    dataFrom:
      - key: "secret/minha-app/config"
    # Adiciona chaves específicas
    data:
      - secretKey: "special-token"
        remoteRef:
          key: "secret/shared/tokens"
          property: "service-token"
```
