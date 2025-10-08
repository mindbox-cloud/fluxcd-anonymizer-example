# Prometheus CRD Deployment via Flux

Этот каталог содержит конфигурацию для деплоя только CRD от Prometheus оператора через Flux CD.

## Что деплоится

- **Prometheus Operator** - только для установки CRD
- **CRD** - Custom Resource Definitions для мониторинга

## Отключенные компоненты

- **Prometheus** - отключен
- **AlertManager** - отключен  
- **Grafana** - отключен
- **Node Exporter** - отключен
- **Kube State Metrics** - отключен

## CRD которые будут доступны

После деплоя будут доступны следующие Custom Resource Definitions:
- `PodMonitor` - для мониторинга подов
- `ServiceMonitor` - для мониторинга сервисов
- `PrometheusRule` - для настройки правил алертинга
- `AlertmanagerConfig` - для настройки AlertManager
- `Probe` - для мониторинга внешних эндпоинтов

## Конфигурация

- **Namespace**: prometheus
- **Ресурсы**: минимальные (64-128Mi RAM)
- **Назначение**: только установка CRD для Helm оператора Flux
