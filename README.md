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
