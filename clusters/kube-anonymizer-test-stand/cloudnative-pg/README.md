# CloudNative-PG Barman Cloud Plugin

Этот каталог содержит конфигурацию для barman-cloud плагина CloudNative-PG без использования cert-manager.

## Что было сделано

### Проблема
По умолчанию barman-cloud плагин требует cert-manager для генерации TLS сертификатов для mTLS связи между CloudNative-PG оператором и плагином.

### Решение
Мы настроили barman-cloud плагин с предварительно сгенерированными самоподписанными сертификатами, что избавляет от необходимости устанавливать cert-manager в тестовом стенде.

## Файлы

- `plugin-barman-cloud.yaml` - основной манифест плагина с TLS конфигурацией
- `barman-cloud-server-secret.yaml` - TLS секрет для сервера
- `barman-cloud-client-secret.yaml` - TLS секрет для клиента

## Как развернуть

1. Примените все манифесты:
```bash
kubectl apply -f .
```

2. Убедитесь, что плагин запущен:
```bash
kubectl get pods -n cloudnative-pg-operator
kubectl get svc -n cloudnative-pg-operator
```

## Альтернатива: Использование cert-manager

Если вы хотите использовать cert-manager вместо предварительно сгенерированных сертификатов:

### 1. Установите cert-manager
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

### 2. Замените секреты на Certificate ресурсы

В `plugin-barman-cloud.yaml` замените секреты на:

```yaml
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: barman-cloud-client
  namespace: cloudnative-pg-operator
spec:
  commonName: barman-cloud-client
  duration: 2160h
  isCA: false
  issuerRef:
    group: cert-manager.io
    kind: Issuer
    name: selfsigned-issuer
  renewBefore: 360h
  secretName: barman-cloud-client-tls
  usages:
  - client auth
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: barman-cloud-server
  namespace: cloudnative-pg-operator
spec:
  commonName: barman-cloud
  dnsNames:
  - barman-cloud
  duration: 2160h
  isCA: false
  issuerRef:
    group: cert-manager.io
    kind: Issuer
    name: selfsigned-issuer
  renewBefore: 360h
  secretName: barman-cloud-server-tls
  usages:
  - server auth
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: cloudnative-pg-operator
spec:
  selfSigned: {}
```

### 3. Удалите предварительно сгенерированные секреты
```bash
kubectl delete -f barman-cloud-server-secret.yaml
kubectl delete -f barman-cloud-client-secret.yaml
```

## Проверка работы

После развертывания проверьте, что CloudNative-PG оператор видит плагин:

```bash
kubectl logs -n cloudnative-pg-operator deployment/cloudnative-pg-controller-manager | grep -i plugin
```

В логах должно быть сообщение о том, что плагин `barman-cloud.cloudnative-pg.io` обнаружен и готов к использованию.

## Использование в PostgreSQL кластере

В манифесте PostgreSQL кластера укажите плагин:

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: my-postgres-cluster
spec:
  # ... другие параметры ...
  plugins:
  - name: barman-cloud.cloudnative-pg.io
    isWALArchiver: true
    parameters:
      barmanObjectName: my-backup-store
```

## Troubleshooting

### Плагин не обнаружен
Если CloudNative-PG оператор не видит плагин, проверьте:

1. **Аннотации в Service**:
```yaml
metadata:
  annotations:
    cnpg.io/pluginClientSecret: barman-cloud-client-tls
    cnpg.io/pluginPort: "9090"
    cnpg.io/pluginServerSecret: barman-cloud-server-tls
  labels:
    cnpg.io/pluginName: barman-cloud.cloudnative-pg.io
```

2. **Аннотации в Deployment**:
```yaml
metadata:
  annotations:
    cnpg.io/pluginName: barman-cloud.cloudnative-pg.io
```

3. **TLS аргументы в deployment**:
```yaml
args:
  - operator
  - --server-cert=/server/tls.crt
  - --server-key=/server/tls.key
  - --client-cert=/client/tls.crt
  - --server-address=:9090
  - --leader-elect
  - --log-level=debug
```

### Перезапуск оператора
CloudNative-PG не обнаруживает плагины динамически. После развертывания нового плагина перезапустите оператор:

```bash
kubectl rollout restart deployment/cloudnative-pg-controller-manager -n cloudnative-pg-operator
```
