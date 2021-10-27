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
