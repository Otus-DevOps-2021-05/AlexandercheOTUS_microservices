# AlexandercheOTUS_microservices
AlexandercheOTUS microservices repository
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №12-13
#### "Установка Docker, запуск контейнера на локальной машине, выполнение команд внутри контейнера, создание образа контейнера на основе запущенного". 
#### "Запуск VM с установленным Docker Engine при помощи Docker Machine. Написание Dockerfile и сборка образа с тестовым приложением. Сохранение образа на DockerHub"
##### Docker установлен локально и на виртуалке в yandex cloud, контейнеры запускаются, создан контейнер reddit, запушен на docker hub.
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №14 
#### "Разбиение приложения на несколько микросервисов. Выбор базового образа. Подключение volume к контейнеру" 
##### Поставил hadolint. Распаковал архив с подготовленными компонентами приложения. Dockerfile для post-py, comment и ui. Сборка контейнеров. Запуск контейнеров в bridge-сети с именем reddit. Оптимизация Dockerfile для ui. Запуск контейнера БД с volume reddit_db. hadolint-проверка всех Dockerfile и замена в них ADD на COPY.
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №15
#### "Практика работы с основными типами Docker сетей. Декларативное описание Docker инфраструктуры при помощи Docker Compose"
##### Запустил контейнер joffotron/docker-net-tools в сетях none и host. Запустил контейнеры проекта в bridge-сети reddit (без network-alias не работает). Запустил контейнеры проекта в 2х bridge-сетях (back_net для данных и front_net для web-сервера). Поставил docker-compose. Собрал и запустил контейнеры проекта из docker-compose.yml. Параметризировал переменными окружения docker-compose.yml (пример в .env.example).
##### Префиксы docker-compose делает по имени каталога, поменять можно по docker-compose --project-name reddit up -d
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №16
#### "Gitlab CI. Построение процесса непрерывной интеграции"
##### Создана виртуалка в яндекс для контейнеров. Запущен контейнер с Gitlab. Gitlab настроен (пароль root, отключена регистрация, добавлены группа и проект). Запущен контейнере с runner. Склонирован код приложения reddit. Пайплайн описан в .gitlab-ci.yml: build-test-deploy, build-test-review (в dev-среду и вручную в stage и prod), deploy в stage и prod по git tag, динамические env для каждой ветки.
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №17
#### "Создание и запуск системы мониторинга Prometheus"
##### Создана виртуалка в яндекс для контейнеров. Запущен контейнер с prometheus. Изменена структура файлов и каталогов. Добавлены Dockerfile для контейнера prometheus и конфиг prometheus.yml с endpoint-ами. Собран контейнер prometheus. В каталоге /src собраны скриптами docker_build.sh контейнеры с ui, post-py и comment. В docker-compose.yml добавлен prometheus, контейнеры с сервисами подняты. В docker-compose.yml добавлен node-exporter, в prometheus.yml добавлен для него endpoint. Контейнер prometheus пересобран, docker-compose перезапущен. Контейнеры запушены в dockerhub: [ui](https://hub.docker.com/repository/docker/paradoxalien/ui), [comment](https://hub.docker.com/repository/docker/paradoxalien/comment), [post](https://hub.docker.com/repository/docker/paradoxalien/post) и [prometheus](https://hub.docker.com/repository/docker/paradoxalien/prometheus).
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №18
#### "Логирование приложений"
##### Создал виртуалку в яндекс для контейнеров. Обновил /src из https://github.com/express42/reddit/tree/logging Пересобрал контейнеры ui, comment, post с тэгом logging и запушил их на докерхаб. Добавил docker-compose-logging.yml с fluentd, elasticsearch и kibana. Добавил Dockerfile и конфиг fluentd, собрал контейнер fluentd. Добавил в docker-compose.yml сервисам post и ui драйвер логирования fluentd. В Kibana сделал индекс для fluentd. В fluent.conf добавил фильтр для структурированных логов от post (json), фильтр для неструктурированных логов от ui (регулярками, потом через Grok-шаблоны). В docker-compose-logging.yml добавил zipkin для трейса логов и включил для контейнеров в docker-compose.yml.
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №19
#### "Установка и настройка Kubernetes"
```
k8s-master в Yandex cloud:
yc compute instance create \
  --name yc-k8s-master \
  --memory 4GB \
  --cores 4 \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/_github/keys/yandex_cloud/appuser.pub

k8s-worker в Yandex cloud:
yc compute instance create \
  --name yc-k8s-worker \
  --memory 4GB \
  --cores 4 \
  --zone ru-central1-a \
  --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
  --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
  --ssh-key ~/_github/keys/yandex_cloud/appuser.pub

Установка docker на обе 2 виртуалки:
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
sudo echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
apt-cache madison docker-ce
sudo apt-get install docker-ce=5:19.03.15~3-0~ubuntu-bionic docker-ce-cli=5:19.03.15~3-0~ubuntu-bionic containerd.io

Установка kubeadm на обе виртуалки:
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
apt-cache madison kubeadm
sudo apt-get install  kubeadm=1.19.15-00 kubectl=1.19.15-00 kubelet=1.19.15-00

На k8s-master:
kubeadm init --apiserver-cert-extra-sans=178.154.231.215 --apiserver-advertise-address=0.0.0.0 --control-plane-endpoint=178.154.231.215 --pod-network-cidr=10.244.0.0/16
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
The recommended driver is "systemd": https://kubernetes.io/docs/setup/production-environment/container-runtimes/

На k8s-worker:
sudo kubeadm join 178.154.231.215:6443 --token ef86y8.8g8bdqwotsum1o4z \
    --discovery-token-ca-cert-hash sha256:65b5e6026d70ecd2313e4571fcf7289d0db44f8df7fd6304fd4329a61137afd5

На k8s-master:
kubectl get nodes
    NAME                   STATUS     ROLES    AGE     VERSION
    fhmh4h9e8a52ar9cdem7   NotReady   <none>   9m27s   v1.19.15
    fhmhpdpj2o71npv49cn0   NotReady   master   21m     v1.19.15
kubectl describe fhmh4h9e8a52ar9cdem7
    reason:NetworkPluginNotReady
curl https://docs.projectcalico.org/manifests/calico.yaml -O
nano calico.yaml
    - name: CALICO_IPV4POOL_CIDR
      value: "10.244.0.0/16"
kubectl apply -f calico.yaml
kubectl label node fhmh4h9e8a52ar9cdem7 node-role.kubernetes.io/worker=worker
kubectl get node
    NAME                   STATUS   ROLES    AGE   VERSION
    fhmh4h9e8a52ar9cdem7   Ready    worker   38m   v1.19.15
    fhmhpdpj2o71npv49cn0   Ready    master   50m   v1.19.15

Запуск post-deployment.yml на k8s-master:
kubectl apply -f post-deployment.yml
На k8s-worker появился контейнер из chromko/post:
docker ps
    CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS               NAMES
    9042c6350b65        chromko/post              "python post_app.py"     9 seconds ago       Up 5 seconds                            k8s_post_post-deployment-799c77ffb-pwv6h_default_94e027b4-3df0-45f5-a1b9-c21daffea8cf_0
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №20
#### "Установка и настройка yandex cloud Kubernetes Engine, настройка локального профиля администратора для yandex cloud. Работа с с контроллерами: StatefulSet, Deployment, DaemonSet"
##### 1. Установка minikube:
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
##### 2. Запуск minikube:
```
$ minikube start --kubernetes-version 1.19.7
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   73s   v1.19.7
$ kubectl config current-context
minikube
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default
```
##### 3. Проверка ui-deployment:
```
$ kubectl apply -f ui-deployment.yml
deployment.apps/ui created
$ kubectl get deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
ui     3/3     3            3           92s
$ kubectl get pods --selector component=ui
NAME                  READY   STATUS    RESTARTS   AGE
ui-75455885d4-rdhsd   1/1     Running   0          2m29s
ui-75455885d4-tm9lw   1/1     Running   0          2m29s
ui-75455885d4-xsg2w   1/1     Running   0          2m29s
$ kubectl port-forward ui-75455885d4-rdhsd 8080:9292
Forwarding from 127.0.0.1:8080 -> 9292
```
##### 4. Проверка comment-deployment:
```
$ kubectl apply -f comment-deployment.yml
deployment.apps/comment created
$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
comment   3/3     3            3           81s
ui        3/3     3            3           13m
$ kubectl get pods --selector component=comment
NAME                      READY   STATUS    RESTARTS   AGE
comment-f86d67d48-2bwx2   1/1     Running   0          2m
comment-f86d67d48-2ctqq   1/1     Running   0          2m
comment-f86d67d48-p9qwm   1/1     Running   0          2m
$ kubectl port-forward comment-f86d67d48-2bwx2 8080:9292
Forwarding from 127.0.0.1:8080 -> 9292
```
##### 5. Проверка post-deployment:
```
$ kubectl apply -f post-deployment.yml
deployment.apps/post created
$ watch kubectl get deployment
$ kubectl get pods --selector component=post
NAME                    READY   STATUS    RESTARTS   AGE
post-78db95568b-9ttls   1/1     Running   0          115s
post-78db95568b-cx2vh   1/1     Running   0          115s
post-78db95568b-l468p   1/1     Running   0          115s
$ kubectl port-forward post-78db95568b-9ttls 8080:5000
Forwarding from 127.0.0.1:8080 -> 5000
```
##### 6. Проверка приложения:
```
$ kubectl apply -f kubernetes/reddit/
deployment.apps/comment created
service/comment created
deployment.apps/mongo created
deployment.apps/post created
deployment.apps/ui created
$ kubectl describe service comment | grep Endpoints
Endpoints:         172.17.0.3:9292,172.17.0.5:9292,172.17.0.6:9292
$ nano kubernetes/reddit/post-service.yml
$ nano kubernetes/reddit/mongodb-service.yml
$ kubectl delete -f kubernetes/reddit/
$ kubectl apply -f kubernetes/reddit/
$ kubectl port-forward ui-75455885d4-x48kd 9292:9292
Forwarding from 127.0.0.1:9292 -> 9292
```
##### 7. Для подключения к mongodb из comment и post:
```
Отредактированы post-service.yml, mongo-deployment.yml, comment-deployment.yml
Добавлены comment-mongodb-service.yml, post-mongodb-service.yml
```
##### 8. Запуск приложения в minikube:
```
$ kubectl delete -f kubernetes/reddit/
$ kubectl apply -f kubernetes/reddit/
$ watch kubectl get deployment
$ kubectl get pods
$ kubectl port-forward ui-75455885d4-wpfz9 9292:9292
Forwarding from 127.0.0.1:9292 -> 9292
http://localhost:9292/
```
##### 9. Добавляем ui-service.yml
```
$ minikube service ui
http://127.0.0.1:37235
```
##### 10. Аддоны (dashboard и metrics-server):
```
$ minikube addons list
$ minikube addons enable dashboard
$ minikube addons enable metrics-server
$ kubectl get namespaces
$ kubectl get all -n kubernetes-dashboard
$ minikube service kubernetes-dashboard -n kubernetes-dashboard
http://127.0.0.1:35443
```
##### 11. Dev-среда:
```
Добавлен dev-namespace.yml
$ kubectl apply -f dev-namespace.yml
$ kubectl apply -n dev -f .
$ minikube service ui -n dev
http://127.0.0.1:34701
Mod ui-deployment.yml
$ kubectl apply -f ui-deployment.yml -n dev
http://127.0.0.1:35029
"Microservices Reddit in dev ui-7f94cb5448-248kt container"
```
##### 12. Kuber-кластер в Yandex Cloud:
```
$ yc managed-kubernetes cluster get-credentials reddit-kuber-cluster --external
$ kubectl cluster-info --kubeconfig /home/avc/.kube/config
Kubernetes control plane is running at https://62.84.116.121
CoreDNS is running at https://62.84.116.121/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://62.84.116.121/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
$ kubectl config current-context
yc-reddit-kuber-cluster
$ kubectl apply -f kubernetes/reddit/dev-namespace.yml
namespace/dev created
$ kubectl apply -f kubernetes/reddit/ -n dev
deployment.apps/comment created
service/comment-db created
service/comment created
namespace/dev unchanged
deployment.apps/mongo created
service/mongodb created
deployment.apps/post created
service/post-db created
service/post created
deployment.apps/ui created
service/ui created
$ kubectl get pods -n dev
NAME                       READY   STATUS    RESTARTS   AGE
comment-76d55f78cd-48rsr   1/1     Running   0          19m
comment-76d55f78cd-7b8kq   1/1     Running   0          19m
comment-76d55f78cd-7pb5k   1/1     Running   0          19m
mongo-6b9fcfd49f-x7x8w     1/1     Running   0          19m
post-d577d7bcb-hjp9b       1/1     Running   0          19m
post-d577d7bcb-khp2p       1/1     Running   0          19m
post-d577d7bcb-l4ptp       1/1     Running   0          19m
ui-7f94cb5448-bmpx4        1/1     Running   0          19m
ui-7f94cb5448-c88qt        1/1     Running   0          19m
ui-7f94cb5448-mvh4p        1/1     Running   0          19m
$ kubectl get nodes -o wide
NAME                        STATUS   ROLES    AGE   VERSION    INTERNAL-IP   EXTERNAL-IP       OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
cl1ff70b8sbpa5um36ru-olur   Ready    <none>   82m   v1.19.15   10.128.0.19   178.154.203.199   Ubuntu 20.04.3 LTS   5.4.0-81-generic   docker://20.10.8
cl1ff70b8sbpa5um36ru-oxiq   Ready    <none>   82m   v1.19.15   10.128.0.25   178.154.200.220   Ubuntu 20.04.3 LTS   5.4.0-81-generic   docker://20.10.8
$ kubectl describe service ui -n dev | grep NodePort
Type:                     NodePort
NodePort:                 <unset>  32092/TCP
```
##### Ссылки на приложение:
##### http://178.154.200.220:32092/
##### http://178.154.203.199:32092/
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №21
#### "Настройка балансировщиков нагрузки в Kubernetes и SSL­Terminating"
##### 1. Селекторные (pod выбирается автоматически) сервисы типа ClusterIP (доступен только изнутри кластера):
```
$ kubectl get service -n dev
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
comment      ClusterIP   10.96.249.42    <none>        9292/TCP         2d6h
comment-db   ClusterIP   10.96.130.58    <none>        27017/TCP        2d6h
mongodb      ClusterIP   10.96.234.23    <none>        27017/TCP        2d6h
post         ClusterIP   10.96.159.34    <none>        5000/TCP         2d6h
post-db      ClusterIP   10.96.241.232   <none>        27017/TCP        2d6h
ui           NodePort    10.96.137.21    <none>        9292:32092/TCP   2d6h
```
##### 2. Погасим dns:
```
$ kubectl scale deployment --replicas 0 -n kube-system kube-dns-autoscaler - сервис следит kube-dns
$ kubectl scale deployment --replicas 0 -n kube-system coredns
$ kubectl get pods -A
$ kubectl exec -ti -n dev post-d577d7bcb-hjp9b ping comment
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
ping: bad address 'comment'
command terminated with exit code 1
```
##### 3. Вернем обратно dns:
```
$ kubectl scale deployment --replicas 1 -n kube-system kube-dns-autoscaler
$ kubectl get pods -A | grep dns
kube-system   coredns-7bc8cf4789-7r64j                 1/1     Running   0          34s
kube-system   coredns-7bc8cf4789-jhzsm                 1/1     Running   0          34s
kube-system   kube-dns-autoscaler-598db8ff9c-hfd76     1/1     Running   0          39s
http://178.154.200.220:32092/ и http://178.154.200.220:32092/ - работают
```
##### 4. Добавим LoadBalancer:
```
$ nano kubernetes/reddit/ui-service.yml
$ kubectl apply -f kubernetes/reddit/ui-service.yml -n dev
$ kubectl get service  -n dev --selector component=ui
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
ui     LoadBalancer   10.96.199.82   178.154.240.246   80:32092/TCP   110s
http://178.154.240.246/
```
##### 5. Добавим Ingress (и уберем LoadBalancer):
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.34.1/deploy/static/provider/cloud/deploy.yaml
$ nano kubernetes/reddit/ui-ingress.yml
$ kubectl apply -f kubernetes/reddit/ui-ingress.yml -n dev
$ nano kubernetes/reddit/ui-service.yml
$ kubectl apply -f kubernetes/reddit/ui-service.yml -n dev
$ kubectl get ingress -A
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAMESPACE   NAME   CLASS    HOSTS   ADDRESS         PORTS   AGE
dev         ui     <none>   *       62.84.114.102   80      2m46s
http://62.84.114.102/
```
##### 6. Добавим SSL:
```
$ nano kubernetes/reddit/ui-ingress.yml
$ kubectl apply -f kubernetes/reddit/ui-ingress.yml -n dev
$ kubectl get ingress -n dev
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME   CLASS    HOSTS   ADDRESS         PORTS   AGE
ui     <none>   *       62.84.114.102   80      78s
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=62.84.114.102"
$ kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
$ kubectl describe secret ui-ingress -n dev
$ nano kubernetes/reddit/ui-ingress.yml
$ kubectl delete ingress ui -n dev
$ kubectl apply -f kubernetes/reddit/ui-ingress.yml -n dev
https://62.84.114.102/
```
##### 7. Добавим NetworkPolicy (mongodb только для post и comment:
```
$ nano kubernetes/reddit/mongo-network-policy.yml
$ kubectl apply -f kubernetes/reddit/mongo-network-policy.yml -n dev
https://62.84.114.102/
```
##### 8. Добавим хранилище для mongodb:
```
$ yc compute disk create --name k8s --size 4 --description "disk for k8s"
$ yc compute disk list
$ nano kubernetes/reddit/mongo-volume.yml
$ nano kubernetes/reddit/mongo-volume-claim.yml
$ nano kubernetes/reddit/mongo-deployment.yml
$ kubectl delete -f kubernetes/reddit/mongo-deployment.yml -n dev
$ kubectl apply -f kubernetes/reddit/mongo-volume.yml
$ kubectl apply -f kubernetes/reddit/mongo-volume-claim.yml -n dev
$ kubectl apply -f kubernetes/reddit/mongo-deployment.yml -n dev
Пересоздание mongo не уничтожит посты в приложении
$ kubectl delete -f kubernetes/reddit/mongo-deployment.yml -n dev
$ kubectl apply -f kubernetes/reddit/mongo-deployment.yml -n dev
```
##### https://62.84.114.102/
-----------------------------------------------------------------------------------------------------------------------------------------------------------
#### Домашнее задание №22
#### "Создание Helm Chart’ов для компонент приложения, управление зависимостями Helm"
##### 1. Установка Helm и tiller:
```
https://github.com/helm/helm/releases/tag/v2.17.0
$ wget https://get.helm.sh/helm-v2.17.0-linux-amd64.tar.gz
$ tar -zxvf helm-v2.17.0-linux-amd64.tar.gz
$ sudo cp linux-amd64/helm /usr/local/bin/helm
$ nano kubernetes/tiller.yml
$ kubectl apply -f kubernetes/tiller.yml
$ kubectl get serviceaccounts -n kube-system | grep tiller
$ helm init --service-account tiller
$ kubectl get pods -n kube-system --selector app=helm
```
##### 2. Helm chart для ui:
```
$ cd kubernetes
$ git mv reddit/ui-deployment.yml Charts/ui/templates/deployment.yaml
$ git mv reddit/ui-ingress.yml Charts/ui/templates/ingress.yaml
$ git mv reddit/ui-service.yml Charts/ui/templates/service.yaml
$ nano Charts/ui/Chart.yaml
$ kubectl delete -f Charts/ui/templates/ -n dev
$ kubectl get pods -n dev | grep ui
$ cd Charts/
$ helm install --name test-ui-1 ui/
$ helm ls --all
$ helm delete test-ui-1 --purge
$ helm install --namespace dev --name test-ui-1 ui/
$ kubectl get pods -A | grep ui
```
##### 3. Helm chart для ui через переменные:
```
$ nano deployment.yaml
$ nano service.yaml
$ nano ingress.yaml
$ nano values.yaml
$ helm install --name test-ui-2 ui/
$ helm install --name test-ui-3 ui/
$ helm upgrade test-ui-2 ui/
$ helm upgrade test-ui-3 ui/
```
##### 4. Helm chart для ui, post и comment через переменные из values.yaml:
```
$ tree Charts/
Charts/
├── comment
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── post
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── reddit
└── ui
    ├── Chart.yaml
    ├── templates
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   └── service.yaml
    └── values.yaml
```
##### 5. Добавим в templates функцию _helpers.tpl (возвращает то же что и {{ .Release.Name }}-{{ .Chart.Name }}):
```
$ tree Charts/
Charts/
├── comment
│   ├── Chart.yaml
│   ├── templates
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── post
│   ├── Chart.yaml
│   ├── templates
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── values.yaml
├── reddit
└── ui
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   └── service.yaml
    └── values.yaml
```
##### 6. Helm Chart приложения reddit (зависимости, mongo):
```
$ tree reddit/
reddit/
├── Chart.yaml
├── requirements.lock
├── requirements.yaml
└── values.yaml
$ helm dependency update
$ helm search mongo
$ nano requirements.yaml
$ helm dependency update
$ helm install reddit --name reddit-test
$ helm ls
NAME            REVISION        UPDATED                         STATUS          CHART           APP VERSION     NAMESPACE
reddit-test     1               Thu Nov 11 20:08:44 2021        DEPLOYED        reddit-0.1.0                    default
$ kubectl get ingress -A
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAMESPACE   NAME             CLASS    HOSTS   ADDRESS         PORTS   AGE
default     reddit-test-ui   <none>   *       62.84.114.102   80      57s
https://62.84.114.102/
$ helm delete reddit-test --purge
$ nano ui/templates/deployment.yaml
$ nano ui/values.yaml
$ nano reddit/values.yaml
$ helm dependency update ./reddit
$ helm install reddit --name reddit-test
https://62.84.114.102/
```
##### 7. Установка gitlab и настройка:
```
$ helm repo add gitlab https://charts.gitlab.io
$ helm fetch gitlab/gitlab-omnibus --version 0.1.37 --untar
$ cd gitlab-omnibus/
$ nano values.yaml
$ nano templates/gitlab/gitlab-svc.yaml
$ nano templates/gitlab-config.yaml
$ nano templates/ingress/gitlab-ingress.yaml
$ helm install --name gitlab . -f values.yaml
$ kubectl get service -n nginx-ingress nginx
http://gitlab-gitlab
Логин root, пароль otusgitlab, новая группа paradoxalien
CI/CD группы, переменные CI_REGISTRY_USER и CI_REGISTRY_PASSWORD
Новые публичные проекты reddit-deploy, ui, post, comment
```
##### 8. Gitlab CI/CD:
```
$ tree Gitlab_ci/
Gitlab_ci/
├── comment
├── post
├── reddit-deploy
└── ui
$ echo "Gitlab_ci/" >> .gitignore
$ cp -r src/ui/* kubernetes/Gitlab_ci/ui
$ cd kubernetes/Gitlab_ci/ui && git init
$ git remote add gitlab http://gitlab-gitlab/paradoxalien/ui.git
$ git add . && git commit -m "init" && git push gitlab
то же для post и comment
$ cp -r kubernetes/Charts/* kubernetes/Gitlab_ci/reddit-deploy && git push gitlab
$ nano kubernetes/Gitlab_ci/ui/gitlab-ci.yml
$ cd kubernetes/Gitlab_ci/reddit-deploy
$ nano ui/templates/ingress.yaml
$ nano ui/values.yaml
$ git checkout -b feature/3
$ nano ui/gitlab-ci.yml
$ git commit -am "Add review feature"
$ git push gitlab feature/3
$ cp ui/gitlab-ci.yml comment/
$ cp ui/gitlab-ci.yml post/
$ nano reddit-deploy/gitlab-ci.yml
$ nano ui/gitlab-ci.yml
$ git push gitlab
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------
