# Kubernetes Microservices Platform (k3s, Minikube, Ingress, TLS, Monitoring)
### Практический проект по развёртыванию микросервисного приложения в **Kubernetes** — от локального окружения до production-like кластера.

### Поставленные цели
- Освоить базовые концепции Kubernetes и работу с объектами (Pod, Deployment, Service).
- Развернуть локальное окружение Kubernetes с использованием Minikube.
- Написать собственные манифесты для микросервисного приложения.
- Развернуть production-like кластер Kubernetes (k3s) на нескольких узлах.
- Настроить маршрутизацию трафика через Ingress Controller (nginx).
- Реализовать TLS-шифрование с использованием cert-manager.
- Настроить постоянное хранилище данных (PersistentVolume / PersistentVolumeClaim).
- Закрепить навыки работы с системой мониторинга, и использованием helm-чартов.

![preview](./img/grafana-dashboard.png)  

### Инструменты и технологии
- Kubernetes (k3s, Minikube) - оркестрация контейнеров
- Docker - контейнеризация
- Ingress Nginx - маршрутизация входящего трафика
- cert-manager - управление TLS-сертификатами
- Helm - пакетный менеджер Kubernetes
- Prometheus Operator - мониторинг кластера
- Grafana - визуализация метрик
- Vagrant + VirtualBox - создание виртуальной инфраструктуры
- PostgreSQL, RabbitMQ - инфраструктурные сервисы

### Структура проекта
- `report.md` - подробный отчёт проделенной работы с командами и скриншотами
- `example/` - пример исходного манифеста
- `manifests_minikube` - манифесты использованные с minikube
- `manifests_k8s/` - Kubernetes манифесты
- `vagrant/Vagrantfile` - описание виртуальной инфраструктуры
- `img/` - изображения и скриншоты

## Содержание отчета
### Stage 1. Basic Kubernetes (Minikube)
- Развернуто локальное окружение Kubernetes с использованием Minikube.
- Применён готовый манифест микросервисного приложения.
- Изучены базовые объекты Kubernetes (Pod, Deployment, Service).
- Настроен доступ к сервисам через NodePort и port-forward.
- Использован Kubernetes Dashboard для визуализации.

### Stage 2. Custom Manifests
- Созданы собственные манифесты для микросервисного приложения.
- Реализовано хранение конфигурации через ConfigMap.
- Реализовано хранение чувствительных данных через Secrets.
- Настроено взаимодействие сервисов через Kubernetes Service.
- Проверена работоспособность приложения и логирование.

### Stage 3. Kubernetes Cluster (k3s)
- Развернут кластер k3s (1 control-plane + 2 worker) и настроено подключение узлов к кластеру.
- Установлен Ingress Controller (nginx).
- Настроен cert-manager для работы с TLS.
- Создан self-signed ClusterIssuer.
- Выпущен wildcard сертификат для локального домена.
- Настроен Ingress для маршрутизации HTTPS-трафика.
- Настроен PersistentVolume (hostPath).
- Реализован PersistentVolumeClaim.
- Использован nodeAffinity для привязки к узлу.
- Развернуто микросервисное приложение, проведены тесты
- Part 4. Развёртывание приложения
- Проверена доступность через Ingress (HTTPS).

### Stage 4. Monitoring (Prometheus Operator)
- Установлен Helm и добавлен репозиторий prometheus-community.
- Развернут стек kube-prometheus-stack.
- Проверен доступ к Grafana.

### Ограничения
- Secrets хранятся в репозитории (учебный проект).
- hostPath не подходит для production-среды.
- self-signed сертификаты не являются доверенными.
- отсутствует внешний балансировщик нагрузки.

### Выводы
1. Освоил базовые и продвинутые концепции Kubernetes.
2. Реализовал развертывание микросервисного приложения в Kubernetes.
3. Настроил production-like кластер на базе k3s.
4. Реализовал маршрутизацию трафика и TLS-шифрование.
5. Обеспечил постоянное хранение данных.
6. Развернул мониторинг кластера и приложений.

