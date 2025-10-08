# Flux CD Deployment for CloudNativePG with Prometheus

Этот каталог содержит конфигурацию для деплоя CloudNativePG оператора и Prometheus Stack через Flux CD.

## Структура деплоя

### 1. Prometheus CRD (деплоится первым)
- **Namespace**: prometheus
- **Компоненты**: только Prometheus Operator для установки CRD
- **CRD**: PodMonitor, ServiceMonitor, PrometheusRule, AlertmanagerConfig, Probe

### 2. CloudNativePG (деплоится после Prometheus)
- **Namespace**: cloudnative-pg-operator
- **Мониторинг**: PodMonitor включен после деплоя Prometheus
- **Версия**: 0.25.0

## Порядок деплоя

1. **Flux System** - базовая система Flux CD
2. **Prometheus** - оператор мониторинга и CRD
3. **CloudNativePG** - оператор PostgreSQL с мониторингом

## Файлы конфигурации

### Prometheus
- `prometheus/namespace.yaml` - namespace для Prometheus
- `prometheus/helm-repository.yaml` - HelmRepository для Prometheus charts
- `prometheus/helm-release.yaml` - HelmRelease для kube-prometheus-stack
- `prometheus/prometheusrule-cloudnative-pg.yaml` - правила алертинга для CloudNativePG

### CloudNativePG
- `cloudnative-pg/namespace.yaml` - namespace для CloudNativePG
- `cloudnative-pg/helm-repository.yaml` - HelmRepository для CloudNativePG charts
- `cloudnative-pg/helm-release.yaml` - HelmRelease без мониторинга (базовый)
- `cloudnative-pg/helm-release-with-monitoring.yaml` - HelmRelease с включенным PodMonitor

## Использование

После деплоя будут доступны:
- **PodMonitor** для мониторинга подов CloudNativePG
- **PrometheusRule** с алертами для CloudNativePG
- **ServiceMonitor** для мониторинга сервисов
- **AlertmanagerConfig** для настройки алертов

## Переключение на мониторинг

После деплоя Prometheus можно переключить CloudNativePG на версию с мониторингом:
1. Замените `helm-release.yaml` на `helm-release-with-monitoring.yaml` в kustomization
2. Flux автоматически обновит деплой с включенным PodMonitor
