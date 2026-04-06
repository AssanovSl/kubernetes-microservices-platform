# Stage 1. Basic Kubernetes. Основные концепции Kubernetes.  
### Запуск окружения Kubernetes. 
Для того что бы развернуть окружение буду использовать **MiniKube** - это инструмент, который поднимает локальный Kubernetes-кластер прямо на хостовой ОС.  
![install minikube](./img/minikube-install.png)  
Далее запускаю окружение с ограничением памяти в 4gb, командой: `minikube start --memory=4096`:  
![start minikube](./img/minikube-start.png)  
Утилита minikube используя Docker (в качестве драйвера) развернула контейнер, и запустила окружение kubernetes, чему свидетельсвтует вывод:  
![nodes](./img/kubectl-nodes.png) 

### Готовый манифест.  
Перед написанием своих манифестов подобрал подходящий пример манифеста описывающий 8 ресурсов (4 Deployment и 4 Service) - `example/microservices.yml` применил этот манифест и проверил запущенные поды:  
![example manifest](./img/manifest-apply.png)  
А так же команда `kubectl get all` позволяет посмотреть список pods, services, deployment и replicaset. Что позволяет оценить рабоспособность и состояние кластера.  
![kubectl all](./img/kubectl-all.png)  

### Стандартная панель управления Kubernetes (dashboard).
Dashboard - это визуальный интерфейс для того же, что показывает `kubectl get all`.  
Скриншоты визуализации дашборда:  
![Dashboard](./img/minikube-dashboard.png)  
![Dashboard](./img/minikube-dashboard-deployment.png)  
![Dashboard](./img/minikube-dashboard-pods.png)

### Доступ к развернутым сервисам через туннель.
Команда `minikube service` позволяет прокинуть туннель от сервиса на хост. (пока команда работает в терминале).  
В этот момент:  
**Minikube** находит NodePort, который проброшен на кластерный под (указанный в команде)  
Создаёт туннель от хостовой ос к кластеру, потому что в Minikube Docker-драйвер создаёт виртуальный контейнерный хост.  
И открывается URL в браузере.  
Полученный туннель живёт пока открыт терминал.  
![tun minikube](./img/kubectl-service.png)  

### Доступ в браузере к странице приложения (сервис apache).  
Используя предыдущую команду получаю открытую страницу apache, отображающую развернутое приложение:  
![apache service web](./img/apache-service-web.png)  

# Stage 2. My manifest. Написание собственных манифестов.  
### Подготовка манифестов.
Испльзую ранее развернутое в [Docker Swarm](https://github.com/AssanovSl/microservices-project-lab) 
Написал первый манифест (карту конфигурации) `configmap.yml`, описывающий все переменные окружения. Данные о микросервисах, их порты, данные БД.   
![Manifest ConfigMap](./img/ConfigMap-manifest.png)  
Далее создал файл `secrets.yml` в котором хранится чувствительная информация (пароли, токены, пароли и т.д). Из предыдущего приложения вынесем данные авторизации Postgres и Rabbit в этот манифест. А так же добавим ключи межсервисной авторизации, которые сейчас прописаны в коде приложения и опубликованный в docker hub. Является уязвимостью.  
Для хранения секретов буду использовать блок stringData, который позволяет указать данные в привычном виде. Kubernetes автоматически конвертирует их в base64 для безопасного хранения внутри кластера.   
![Manifest Seecrets](./img/Secrets-minifest.png)  
После этого остается добавить манифест описывающий все сервисы предложенного приложения.  
На примере **session-service** составил манифест:  
<details>
<summary>session service manifest</summary>

```yml
apiVersion: apps/v1 
kind: Deployment
metadata:
  labels:
    run: session-service
  name: session-service
spec: 
  replicas: 1
  selector:
    matchLabels:
      run: session-service
  template:
    metadata:
      labels:
        run: session-service
    spec: 
      containers:
      - image: dorothag/session-service:latest 
        name: session-service
        ports:
        - containerPort: 8081
        env:
            # берутся переменные окружения из configMap
          - name: POSTGRES_HOST
            valueFrom:
              configMapKeyRef:
                name: app-config
                key: POSTGRES_HOST
          - name: POSTGRES_PORT
            valueFrom:
              configMapKeyRef:
                name: app-config
                key: POSTGRES_PORT
          - name: POSTGRES_DB
            valueFrom:
              configMapKeyRef:
                name: app-config
                key: POSTGRES_DB_SESSION
          - name: JAVA_OPTS
            valueFrom:
              configMapKeyRef:
                name: app-config
                key: JAVA_OPTS
            # берутся переменные окружения (cекреты) из secrets
          - name: POSTGRES_USER
            valueFrom:
              secretKeyRef:
                name: app-secrets
                key: POSTGRES_USER
          - name: POSTGRES_PASSWORD
            valueFrom:
              secretKeyRef:
                name: app-secrets
                key: POSTGRES_PASSWORD
```
</details>  

Здесь указаны тип создаваемого объекта, метаданные для усправления кубернетиса подами, состав поды и необходимые переменные и порты.  
Здесь используются ранее вынесенные переменные и секреты.  
Аналогично представленному сервису прописал и остальные составляющие приложения.  
Так же добавил каждому сервису объект **Service** что позволяет подам общаться между собой используя имя. а не ip-адрес (который после перезапуска меняется)  
![Manifest Service](./img/service-manifest.png)  

### Последовательный запуск.
Запустил окружение и первые манифесты, configMap и secrets:  
![Kubctl up](./img/kubectl-apply-1.png)  
Далее запускаю БД и очередь:  
![Kubctl up](./img/kubectl-apply-postgres-rabbit.png)  
И наконец запускаю поды приложения:  
![Kubctl up](./img/kubctl-apply-microservice.png)  
![Kubctl up](./img/manifest-up.png)    

### Проверка созданных объектов.  
Получаем список ConfigMap:  
![Get configMap](./img/get-configmap.png)  
Проверка секретов:  
![Get secrets](./img/get-secrets.png)  
Подробная информация скрыват не отображает пароли и ключи.  
Проверка Pod:
![Get pods](./img/get-pods.png)  
Проврека Service:
![Get svc](./img/get-svc.png)  
Итого
- Все **pod** находятся в статусе Running, что говорит об успешном запуске контейнеров.
- **Сервисы** корректно сопоставлены с pod через selector.
- **ConfigMap** и **Secrets** успешно подключаются к контейнерам через переменные окружения.

### Проверка наличие значений секретов.  
Все значения секретов хранятся в закодированном виде, чему свидельствует пвывод первой команды:  
![Secrets code/decode](./img/get-secret-decode.png)  
Далее применял декодирование что бы получить действительный пароль, указанный в secrets.yml. 

### Отображение логов:
Командой `kubectl logs` с указанием конкретного контейнера (поды) посмотрим вывел логи.
![Kuber logs](./img/minikube-logs.png)  
Аналогично посмотрел логи базы данных:  
![Kuber logs](./img/minikbe-logs-2.png)  

### Туннели для доступа к gateway service и session service.
Что бы пробросить порт с поды на хост можно воспользоваться командой `minikube tunnel`. Таким образом присвоится рандомный порт хостовой ос для доступа к поде.
В моем варианте я указывал `type: NodePort` в service, таким образом порт уже проброшен на хост.  
![NodePort in minikube](./img/minikube-nodeport.png)  
Так как тесты в postman подразумевают конкретный порт по localhost, воспользовался **port-forward**  
![Port Forwarding](./img/minikube-port-forwarding.png)
Аналогично можно прокинуть необходимые порты.

### Тесты Postman:
После того как необходимые порты проброшены, можно провести тесты Postman:  
![Postman](./img/postman-tests.png)  
Все тесты пройдены успешно.

### Панель управления Kubernetes:
1 - Текущее состояние узлов кластера:
![Dashboard](./img/minikube-dashboard-1.png)  
2 - Список запущенных Pod:
![Dashboard](./img/minikube-dashboard-pod.png)  
3 - Дополнительные метрики ноды:  
![Dashboard](./img/minikube-dashboard-nodes.png)


# Stage 3. Развертывание и управление кластером Kubernetes.  
## Part 1. Кластер k3s.
Подготовил виртуальные машин с использованием Vagrant и Virtualbox:  
![Vagrantfile](./img/vagrantfile.png)  
(полный и актуальный vagrantfile доступен в репозитории vagrant/Vagrantfile).  

### Установка **k3s** на созданные ВМ:  
В своем клатере отказываюс от работы с ingress traefik в пользуя nginx.  
На ВМ server устанавливаю k3s командой:  
```bash
curl -sfL https://get.k3s.io | sh -s - \
  --disable=traefik \
  --node-name server \
  --node-ip 192.168.56.10 \
  --advertise-address 192.168.56.10 \
  --tls-san 192.168.56.10 \
  --flannel-iface eth1 \
  --write-kubeconfig-mode 644
```
указав при помощи флагов, ip-адрес для сервера, адресс куда подключаются оставшиеся узлы, и дать права для vagrant к kubeconfig.  
![Install k3s](./img/k3s-install-on-server.png) 
`kubectl get nodes` возвращает одну ноду **control-plane** после этого необходимо добавить воркеры, **worker node**. Для этого необходимо сохранить токен.  
![k3s token](./img/k3s-server-token.png)  
Подключившись к узлу worker01 выполнить команду:  
```bash
curl -sfL https://get.k3s.io | K3S_URL="https://192.168.56.10:6443" \
K3S_TOKEN="K104215d9ff47d08171a5bc5d3e9d564c6a8fd3674d79fd43a8b0fd30cca3653ca2::server:c21ee2eb947d7bc0a35132cb9848fc73" sh -s - \
  --node-name worker02 \
  --node-ip 192.168.56.12 \
  --flannel-iface eth1
```
где, `K3S_TOKEN` это токен сформированный на control-plane.  
И аналогично выполнить и на worker02 для подключения.  
![Install k3s](./img/k3s-install-on-worker.png)  
После установки проверил на сервере ноды:  
![k3s nodes](./img/k3s-nodes.png)  
Все ноды подключены к кластеру.  

### Установка **Ingress Controller Nginx**
При установке я отключил стандартный ingress Controller (Traefik) необходимо добавить, установить Nginx.  
Для этого использую официальный манифест из репозитория используя команду:  
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml`  
![Ingress Controller Nginx](./img/ingress-install.png)  
Проверил работу nginx командой `kubectl get pods --namespace ingress-nginx` с указанием необходимого namespace:  
![Ingress pod running](./img/ingress-pod.png)  

### Настройка TLS для развернутого приложения
Конфигурирование внутри кластера утилиты *cert-manager**  
**cert-manager** это Kubernetes-оператор, который автоматически выдаёт и обновляет сертификаты TLS для сервисов внутри кластера. Таким образом будет использоваться для ingress, что бы приложение могло работать по HTTPS (защищенное подключение)
Аналогично ingress буду использовать официальный манифест, и устанавливаю командой:  
`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.20.1/cert-manager.yaml`
![Cert manager](./img/cert-manager-install.png) 
Убедился что все установилось, namespace создан, поды запущены:   
![Cert manager](./img/cert-manager-install-2.png)  
Работают 3 поды:
**cert-manager** управляет сертификатами, отвечает за процесс генерации/обновления. 
**webhook** проверяет правильность конфигурации, результат работы cert-manager.
**cainjector** следит за секретами с сертификатами, контролирует чтобы CA попал в pods

На данном этапе cert-manager может сгенерировать обновить сертификат, но он не будет подписанным, для этого не хватает центра сертификации. Т.к я не привязываю свой домен (использую локальное доменное имя) и буду использовать локальный центр-сертификации.
Для этого необходимо в кластере создать **ClusterIssuer** (т.к Issuer актуален внутри одного namespace) для этого использую манифест:  
```yml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: local-issuer
spec:
  selfSigned: {}
``` 
Применю манифест одной командой:  
![kubectl get clusterissuer](./img/clusterissuer-add.png)  
Создался ClusterIssuer с типом **selfSigned**, который позволяет cert-manager выпускать самоподписанные сертификаты внутри кластера

Теперь можно создать сертификат для приложения:  
В качестве доменного имени использован локальный домен **\*.dorothag.local**.  
```yml
apiVersion: cert-manager.io/v1
kind: Certificate  # тип объекта, заявка на сертификат.
metadata:
  name: dorothag-cert
  namespace: app   # неймспейс для будующего приложения
spec:
  secretName: dorothag-tls  # Имя где будет готовый сертификат
  issuerRef:            # Данные "центра сертификации" 
    name: local-issuer   
    kind: ClusterIssuer
  commonName: dorothag.local
  dnsNames:
    - dorothag.local     # Доменное имя для которого нужен серт.
    - "*.dorothag.local" # wildcard-сертификат! Подходит для любых поддоменов
```
Создал namespace **app** для будущего приложения т так же применю этот манифест одной командой:  
![Cert create](./img/cert-create.png)  
Проверил наличие сертификата и секрета:  
![Get cert](./img/kubectl-get-cert.png)  

В рамках выполнения задания использован wildcard-сертификат для локального домена, сгенерированный с помощью cert-manager и self-signed ClusterIssuer. Данный подход позволяет протестировать работу HTTPS внутри кластера без необходимости использования внешнего центра сертификации. В production-сценариях вместо self-signed сертификатов применяется доверенный CA, например Let's Encrypt.

### Настройка ingress для домена.  
Созданный ранее сертификат будет использоваться для **Ingress** осталось написать манифест `ingress.yml` и применить его:  
Применил и проверил Ingress:  
![Ingress Apply](./img/ingress-apply.png)  
![Ingres Decribe](./img/ingress-descr.png)  
На данный момент сервисы не найдены так как манифест приложения еще не применен.

### Работа с хранилищем (Persistent Volume):  
Поды с kubernetis не вечные, могут падать, перезапускаться, и что бы данные не терялись (не обнулялись) необходимо зранить данные отдельно, вне поды. **Volume** внешнее хранище данных.  
В Kubernetes есть варината хранилищ:  
1. **PersistentVolume** (PV) случай когда сам создаешь хранилище и говоришь поду использовать его.  
2. **PersistentVolumeClaim** (PVC) когла под сам запрашивает хранилище у кластера.  
Использовал PersistentVolume с типом hostPath, указывающий на директорию `/data/postgres` на узле кластера.  
Но пода может подняться и на другой ноде, что бы этого избежать использован механизм **nodeAffinity**, ограничивающий использование PV конкретным узлом.  
Описал **PersistentVolume** с ограничением ноды:  

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 2Gi  
  accessModes:
    - ReadWriteOnce   # только одна нода может писать
  persistentVolumeReclaimPolicy: Retain   # данные не удаляются при удалении PVC
  storageClassName: manual   
  hostPath:
    path: /data/postgres   # путь на ноде (реальный диск)

  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - worker01   # Строго указал worker01 ноду.
```

Далее описал **PersistentVolumeClaim** для запроса подой хранилища:  
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: app
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual   # должен совпадать с PV
  volumeName: postgres-pv   # жёстко указываем PV
  resources:
    requests:
      storage: 1Gi   # Объем памяти
```
Таким образом обеспечивается сохранность данных при перезапуске Pod и корректная привязка к узлу, на котором расположено хранилище.
Используя манифест из прошлого проекта добавляю объекты PV и PVС. И добавил volume использующий PVC в Deployment 
```yml
  kind: Deployment
  ...
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data   # куда postgres пишет данные

      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc   # подключаем PVC
  ...
```

### Запуск приложения в новом кластере.  
Применяю/проверяю ConfigMap - ✅
Применяю Secrets - ✅
Применяю PV, PVC, БД - ✅  
![Kubectk apply volumes](./img/apply-pv-pvc.png)  
Следующимм шагом RabbitMQ - ✅ 
И оставшийся стек приложение с манифестом `microservices.yml` - ✅
Проверяю что бы все запустилось:  
Поды:
![Kubectl apply app](./img/apply-microservices.png)  
Так же проверил сервисы:  
![Kubectl apply app](./img/apply-microservices-svc.png)  
Приложение в кластере kubernetes (k3s) считается развернутым!


### Проврка работоспособности тестами.  
Перед проведеннием тестов добавил А-записись (соответсвите адреса ingress и моего локального домена) и отредактировал адреса запросов, удавил порты и указав свой домен:  
![Hosts+Postman](./img/Postman-hosts.png)  
Все тесты прошли успешно:  
![Postman](./img/Postman-test.png)  
А так же проверил доступность я https с хоста по домену:  
![Https test](./img/https-test.png)  

# Stage 4. Мониторинг Prometheus Operator.   
### Работа с готовыми helm-чартами.  
Добавляю к работающему приложение **мониторинг** Prometheus Operator это инструмент для автоматизированного управления системой мониторинга в Kubernetes.  
Оператор в отличии от обычного Prometheus умеет: автоматически обнаруживает сервисы, управляет конфигурацией Prometheus, упрощает масштабирование и обновление.

Разворачивать **Prometheus Operator** буду с использованием Helm-чарта.  
**Helm-чарт** — это пакет шаблонов Kubernetes-манифестов, который позволяет быстро развернуть сложное приложение.  
Для работы с Helm необходимо установить CLI-инструмент:  
![Install Helm](./img/helm-istall.png)  
Когда установлен helm, нужно добавить официальный репозиторий с мониторингом и обновить список:  
![Add helm repo](./img/helm-add-repo.png)  
Далее перед установкой создаю новый namespace **monitoring**  
И устанавливаю стек мониторинга командой:  
```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```
![Install Monitoring](./img/helm-monitoring-install.png)  
В подсказке вся необходимая информация: пароль для доступа к Grafana, и команда для проброса порта графаны на хост.  
![Prometheus pods](./img/prometheus-get-pods.png)  
Так же проверил доступность дашборда в браузере:  
![Grafana dashboard](./img/grafana-dashboard.png)
