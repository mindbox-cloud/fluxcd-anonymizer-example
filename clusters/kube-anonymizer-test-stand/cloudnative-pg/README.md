# CloudNativePG Deployment via Flux

Этот каталог содержит конфигурацию для деплоя CloudNativePG оператора через Flux CD.

## Структура

- `namespace.yaml` - Namespace для CloudNativePG оператора
- `helm-repository.yaml` - HelmRepository для CloudNativePG charts
- `helm-release.yaml` - HelmRelease для основного оператора
- `kustomization.yaml` - Kustomization для деплоя всех ресурсов
- `values-*.yaml` - Конфигурации для разных окружений

## Использование

Flux автоматически подхватит изменения из Git репозитория и задеплоит CloudNativePG оператор.

## Конфигурация

Основные параметры конфигурации:
- Версия CloudNativePG: 0.25.0
- Namespace: cloudnative-pg-operator
- Мониторинг: включен PodMonitor
- Ресурсы: настраиваются через values файлы

## Окружения

- `values-dev.yaml` - конфигурация для dev окружения (уменьшенные ресурсы)
- `values-staging.yaml` - конфигурация для staging окружения
- По умолчанию используется production конфигурация из `helm-release.yaml`
