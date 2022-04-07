# pavel-strokov_platform
pavel-strokov Platform repository

# Выполнено ДЗ № 1

 - [*] Основное ДЗ
 - [*] Задание со *

## В процессе сделано:
 - Установлена виртуальная машина (Vagrant + Oracle Virtual Box ) с IP адресом 172.31.250.15.
 - В виртуальной машине установлена версия kubernetes minikube.
 - После запуска minikube командой minikube start проверяем её работу командой 
 ```bash
kubectl cluster-info
```
Результат выполнения
```bash
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
Проверяем что произойдёт после выполнения команды (удаление всех подов)
```bash
docker rm -f $(docker ps -a -q)
```
Проверим состояние кластера
```bash
kubectl get cs
```
Результат выполнения
```bash
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
```
Все поды в namespace kube-system восстановились, кроме storage-provisioner

Исследуем состояния pods, чтобы определиьт причину того что они были восстановлены. Для этого воспользуемся командой describe.

```bash
kubectl describe pod pod-name -n kube-system
```
В результате исследования полученных результатов обнаружилось следующее:

Pod **coredns-64897985d-qpvps** Controlled By:  ReplicaSet/coredns-64897985d

Управляется ReplicaSet, соответственно при удалении poda, контротроллер репликации обнаруживает его отсутствие и создаёт новый.

Pod **kube-proxy-mj6r5**                 Controlled By:  DaemonSet/kube-proxy

Управляется DaemonSet, соответственно при удалении poda, он обнаруживает его отсутствие и создаёт новый.

Pods
- **etcd-minikube**                    Controlled By:  Node/minikube
- **kube-apiserver-minikube**          Controlled By:  Node/minikube
- **kube-controller-manager-minikube** Controlled By:  Node/minikube
- **kube-scheduler-minikube**          Controlled By:  Node/minikube

Остальные элементы запускаются и отслеживаются непосредственно виртуальной машиной minikube. 
Можно зайти в minikube по ssh и выполнить команду ps -ax. Все эти процессы будут отображаться в выводе.

Pod **storage-provisioner**              запускается как обычный pod и после удаления не восстанавливается.
Похоже что этот под запускается только при запуске виртуальной машины.

## Далее
- Создан [Dokerfile](./kubernetes-intro/web/Dockerfile), который описывает запуск webserver на порту 8000. Этот webserver отдаёт содержимое файлов , которые находятся в папке [web](./kubernetes-intro/web/public). Изначально там размещены 2 файла [index.html](./kubernetes-intro/web/public/index.html)  и [homework.html](./kubernetes-intro/web/public/homework.html).

Переходим в папку [web](./kubernetes-intro/web/) и cобираем контейнер 
```bash
docker build -t pstrokov/homework:latest .
```
Выкладываем его на dockerhub.com
```bash
docker push pstrokov/homework:latest
```
Создаём файл описания pod для его запуска [web-pod.yaml](./kubernetes-intro/web-pod.yaml)

Запускаем pod командой 
```bash
kubectl apply -f web-pod.yaml
```
Для того, чтобы увидеть результат в браузере на хост-машине выполняем команду 
```bash
kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
```
Смотрим результат по ссылке http://172.31.250.15:8000/homework.html.

Добавляем в описание pod InitContainer, который генерит index.html при запуске пода.

Пересобираем образ и отправляем его в dockerhub.

Перезапускаем pod и наблюдаем результат.

## Задание со *
Клонируем указанный репозиторий в свою виртуальную машину.

Переходим в папку src/frontend.

Запускаем сборку проекта.
```bash
docker build -t pstrokov/hipster-frontend:latest .
```
Публикуем образ на dockerhub
```bash
docker push pstrokov/hipster-frontend:latest
```
Создаём файл опимания [frontend-pod.yaml](./kubernetes-intro/frontend-pod.yaml)

Запускаем 
```bash
kubectl apply -f frontend-pod.yaml
```
Проверяем статус pod
```bash
kubectl get pods
```
Результат не утешающий ....  
```bash
NAME       READY   STATUS    RESTARTS   AGE
frontend   0/1     Error     0          36s
web        1/1     Running   0          17m
```
Для того чтобы определить причину - проверяем log.
```bash
kubectl logs frontend
```
В логе обнаруживаем запись 
```bash
panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
```
Не определена переменная окружения. После изучение по приведённой [ссылке](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/kubernetes-manifests/frontend.yaml) содержимого файла, определяем, что для корректной работы необходимо определить набор переменных окружения.

Создаём новый файл [frontend-pod-healthy.yaml ](./kubernetes-intro/frontend-pod-healthy.yaml), с учётом данных из указанного файла.

Запускаем pod с учётом внесённых изменений.
```bash
kubectl apply -f frontend-pod-healthy.yaml
```
Результат
```bash
NAME       READY   STATUS    RESTARTS   AGE
frontend   1/1     Running   0          8s
web        1/1     Running   0          26m
```

# Выполнено ДЗ № 2

 - [*] Основное ДЗ
 - [*] Задание со *

## В процессе сделано:
- Установлена виртуальная машина (Vagrant + Oracle Virtual Box ) с IP адресом 172.31.250.16.
- Создан yaml файл [kind-config.yaml](./kubernetes-controllers/kind-config.yaml), содержащий описание локального кластера.
- Запущен локальный кластер командой
```bash
kind create cluster --config kind-config.yaml
```
Были созданы 3 control-plain ноды и 3 worker ноды.
```bash
[vagrant@kind kind]$ kubectl get nodes
NAME                  STATUS   ROLES                  AGE    VERSION
kind-control-plane    Ready    control-plane,master   16m    v1.23.4
kind-control-plane2   Ready    control-plane,master   11m    v1.23.4
kind-control-plane3   Ready    control-plane,master   10m    v1.23.4
kind-worker           Ready    <none>                 2m5s   v1.23.4
kind-worker2          Ready    <none>                 2m5s   v1.23.4
kind-worker3          Ready    <none>                 2m5s   v1.23.4
```

### Replicaset
Был создан манифест [frontend-replicaset.yaml](./kubernetes-controller/frontend-replicaset.yaml) из задания, который при запуске выдал ошибку. 

Ошибка:  missing required field "selector" in io.k8s.api.apps.v1.ReplicaSetSpec.

Отсутствует обязательное поле selector в спецификации ReplicaSet.

Необходимое поле было добавлено.
```yaml
selector:
  matchLabels:
    tier: frontend
```
Так же поле tier: frontend было добавлено в метки темплейта.

```bash
kubectl get pods -l app=frontend

NAME             READY   STATUS    RESTARTS   AGE
frontend-x7p8f   1/1     Running   0          5m14s
```

всё заработало ....
Увеличим количество реплик 
```bash
kubectl scale replicaset frontend --replicas=3
```


```bash
replicaset.apps/frontend scaled

NAME             READY   STATUS              RESTARTS   AGE
frontend-h7d5j   0/1     ContainerCreating   0          15s
frontend-t6z2p   0/1     ErrImagePull        0          15s
frontend-x7p8f   1/1     Running             0          6m23s
```

```bash
kubectl get rs frontend

NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6m50s
```

Удалим поды приложения командой и понаблюдаем результат 
```bash
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
```

Результатом будет ....

```bash
NAME             READY   STATUS    RESTARTS   AGE
frontend-h7d5j   1/1     Running   0          101s
frontend-t6z2p   1/1     Running   0          101s
frontend-x7p8f   1/1     Running   0          7m49s
frontend-h7d5j   1/1     Terminating   0          101s
frontend-t6z2p   1/1     Terminating   0          101s
frontend-x7p8f   1/1     Terminating   0          7m50s
frontend-ctjnj   0/1     Pending       0          1s
frontend-7rnrh   0/1     Pending       0          0s
frontend-z64tk   0/1     Pending       0          0s
frontend-ctjnj   0/1     Pending       0          1s
frontend-7rnrh   0/1     Pending       0          0s
frontend-z64tk   0/1     Pending       0          1s
frontend-ctjnj   0/1     ContainerCreating   0          2s
frontend-7rnrh   0/1     ContainerCreating   0          1s
frontend-z64tk   0/1     ContainerCreating   0          1s
frontend-h7d5j   0/1     Terminating         0          104s
frontend-t6z2p   0/1     Terminating         0          104s
frontend-h7d5j   0/1     Terminating         0          104s
frontend-t6z2p   0/1     Terminating         0          104s
frontend-x7p8f   0/1     Terminating         0          7m52s
frontend-h7d5j   0/1     Terminating         0          104s
frontend-t6z2p   0/1     Terminating         0          104s
frontend-x7p8f   0/1     Terminating         0          7m53s
frontend-x7p8f   0/1     Terminating         0          7m53s
frontend-ctjnj   1/1     Running             0          5s
frontend-7rnrh   1/1     Running             0          5s
frontend-z64tk   1/1     Running             0          5s
```

ReplicaSet восстановила нужное количество реплик приложения.

Применим обратно созданный ранее манифест.

```bash
kubectl apply -f frontend-replicaset.yaml
```

```bash
NAME             READY   STATUS    RESTARTS   AGE
frontend-ctjnj   1/1     Running   0          2m28s
```

осталась одна работающая реплика ....

Поменяем поле replicas в файле на 3

Применим манифест

получим 3 получим 3 работающих пода

```bash
NAME             READY   STATUS    RESTARTS   AGE
frontend-7sfk8   1/1     Running   0          7s
frontend-ctjnj   1/1     Running   0          23m
frontend-nxthq   1/1     Running   0          7s
```

Перетегируем наш ранее собранный контейнер на версию v0.0.2 и обновим манифест.

Применим его

```bash
kubectl apply -f frontend-replicaset.yaml | kubectl get pods -l app=frontend -w
```

```bash
NAME             READY   STATUS    RESTARTS   AGE
frontend-ctjnj   1/1     Running   0          11m
```

```bash
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
вернул  pstrokov/hipster-frontend:v0.0.2 - новая версия

```bash
kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
```

вернул pstrokov/hipster-frontend:latest

После удаления подов получим 
```bash
NAME             READY   STATUS    RESTARTS   AGE
frontend-h79pt   1/1     Running   0          5m26s
frontend-l67b4   1/1     Running   0          5m7s
frontend-ntjgc   1/1     Running   0          5s
```

Новые поды работают с образом pstrokov/hipster-frontend:v0.0.2

Создаём деплоймент [frontend-deployment.yaml](./kubernetes-controller/frontend-deployment.yaml) и применяем его
```bash
kubectl apply -f paymentservice-deployment.yaml
```

```bash
NAME                       READY   STATUS    RESTARTS   AGE
payment-79cfb49587-jmgwq   1/1     Running   0          112s
payment-79cfb49587-q9hwq   1/1     Running   0          112s
payment-79cfb49587-swjgg   1/1     Running   0          112s
```

Меняем образ на версию v0.0.2

```bash
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
```
Результат выполнения команды

```bash
NAME                       READY   STATUS    RESTARTS   AGE
payment-79cfb49587-c9gpm   1/1     Running   0          2m30s
payment-79cfb49587-cbwt7   1/1     Running   0          2m28s
payment-79cfb49587-mgpvv   1/1     Running   0          2m33s
payment-56f7c49bf8-56265   0/1     Pending   0          0s
payment-56f7c49bf8-56265   0/1     Pending   0          0s
payment-56f7c49bf8-56265   0/1     ContainerCreating   0          0s
payment-56f7c49bf8-56265   1/1     Running             0          5s
payment-79cfb49587-c9gpm   1/1     Terminating         0          2m36s
payment-56f7c49bf8-k4qqv   0/1     Pending             0          0s
payment-56f7c49bf8-k4qqv   0/1     Pending             0          0s
payment-56f7c49bf8-k4qqv   0/1     ContainerCreating   0          0s
payment-56f7c49bf8-k4qqv   1/1     Running             0          2s
payment-79cfb49587-cbwt7   1/1     Terminating         0          2m36s
payment-56f7c49bf8-csvtc   0/1     Pending             0          0s
payment-56f7c49bf8-csvtc   0/1     Pending             0          0s
payment-56f7c49bf8-csvtc   0/1     ContainerCreating   0          0s
payment-56f7c49bf8-csvtc   1/1     Running             0          2s
payment-79cfb49587-mgpvv   1/1     Terminating         0          2m43s
```


Создаются новые поды с новой версией, а старые поды удаляются.

В конце остаётся 3 работающих пода.

И есть 2 ReplicaSet 

```bash
NAME                 DESIRED   CURRENT   READY   AGE
payment-56f7c49bf8   3         3         3       4m10s
payment-79cfb49587   0         0         0       7m26s
```

Одна из них ничем не управляет (с большим временем жизни - т.е. та которая была развёрнута первой).
История
```bash
kubectl rollout history deployment payment
```

```bash
deployment.apps/payment 
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
```

Делаем откат

```bash
kubectl rollout undo deployment paymentservice --to-revision=3 | kubectl get rs -l app=paymentservice -w
```

После этого  картина меняется

```bash
NAME                 DESIRED   CURRENT   READY   AGE
payment-56f7c49bf8   0         0         0       7m22s
payment-79cfb49587   3         3         3       10m
```

Работают поды, развёрнутые из старого деплоймента (большее время жизни)

### Deployment
Для начала собираем две версии образа paymentService и размещаем их на DockerHub.

И создаём файл [paymentservice-replicaset.yaml](./kubernetes-controllers/paymentservice-replicaset.yaml) аналогично созданному в предыдущем ДЗ [frontend-pod.yaml](./kubernetes-intro/frontend-pod.yaml).

Проверяем его работоспособность.

После этого копируем его в файл [paymentservice-deployment.yaml](./kubernetes-controllers/paymentservice-deployment.yaml)

Поле kind меняем с ReplicaSet на Deployment и применяем манифест

```bash
kubectl apply -f paymentservice-deployment.yaml
```
Наблюдаем 3 Replicaset

$ kubectl get deployments
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
payment   3/3     3            3           2m15s

И Deployment

```bash
$ kubectl get rs
NAME                 DESIRED   CURRENT   READY   AGE
payment-56f7c49bf8   3         3         0       51s
```

Обновляем версию в манифесте на v0.0.2

```bash
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
NAME                       READY   STATUS    RESTARTS   AGE
payment-56f7c49bf8-86m2z   1/1     Running   0          7m29s
payment-56f7c49bf8-hfddw   1/1     Running   0          7m29s
payment-56f7c49bf8-hzg4j   1/1     Running   0          7m29s
payment-79cfb49587-m7mlq   0/1     Pending   0          0s
payment-79cfb49587-m7mlq   0/1     Pending   0          0s
payment-79cfb49587-m7mlq   0/1     ContainerCreating   0          0s
payment-79cfb49587-m7mlq   1/1     Running             0          4s
payment-56f7c49bf8-hzg4j   1/1     Terminating         0          7m35s
payment-79cfb49587-shqz4   0/1     Pending             0          0s
payment-79cfb49587-shqz4   0/1     Pending             0          0s
payment-79cfb49587-shqz4   0/1     ContainerCreating   0          0s
payment-79cfb49587-shqz4   1/1     Running             0          37s
payment-56f7c49bf8-86m2z   1/1     Terminating         0          8m12s
payment-56f7c49bf8-hzg4j   0/1     Terminating         0          8m12s
payment-56f7c49bf8-hzg4j   0/1     Terminating         0          8m13s
payment-79cfb49587-q85mc   0/1     Pending             0          1s
payment-79cfb49587-q85mc   0/1     Pending             0          1s
payment-56f7c49bf8-hzg4j   0/1     Terminating         0          8m13s
payment-79cfb49587-q85mc   0/1     ContainerCreating   0          1s
payment-79cfb49587-q85mc   1/1     Running             0          4s
payment-56f7c49bf8-hfddw   1/1     Terminating         0          8m16s
```
Старые поды умерли, новые запустились.

История изменений 

```bash
kubectl rollout history deployment paymentservice
deployment.apps/paymentservice 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```
Делаем откат 
```bash
kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
NAME                 DESIRED   CURRENT   READY   AGE
payment-56f7c49bf8   0         0         0       14m
payment-79cfb49587   3         3         3       6m33s
```

Реализуем 2 стратегии развёртывания в соответствии с заданием.
Создаём два файла Аналог blue-green: [paymentservice-deployment-bg.yaml](./kubernetes-controllers/paymentservice-deployment-bg.yaml)  и Reverse Rolling Update: [paymentservice-deployment-reverse.yaml](./kubernetes-controllers/paymentservice-deployment-reverse.yaml).

Проверяем и убеждаемся, что файлы обеспечивают необходимую стратегию развёртывания.

### Probes
Создаём [frontend-deployment.yaml](./kubernetes-controller/frontend-deployment.yaml) в соответствии с методичкой.

Добавляем readinessProbe. И применяем манифест.
```bash
kubectl apply -f frontend-deployment.yaml 
kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
frontend-5c949d9d77-7f9c8   1/1     Running   0          22s
frontend-5c949d9d77-pr2ks   1/1     Running   0          22s
frontend-5c949d9d77-tqbm5   1/1     Running   0          22s
```

Если поменять url на несуществующий, то получим следующее.

```bash
NAME                        READY   STATUS    RESTARTS   AGE
frontend-6f7b786ccf-nfdhb   0/1     Running   0          30s
frontend-f44dbfdb6-7t82k    1/1     Running   0          104s
frontend-f44dbfdb6-bbfbz    1/1     Running   0          119s
frontend-f44dbfdb6-jh846    1/1     Running   0          83s
```
Новый под не может запуститься. Посмотрим причину.
```bash
kubectl describe pod frontend-6f7b786ccf-nfdhb
```
Причина в следующем
```bash
...
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m1s                default-scheduler  Successfully assigned default/frontend-6f7b786ccf-nfdhb to kind-worker2
  Normal   Pulled     2m1s                kubelet            Container image "pstrokov/hipster-frontend:v0.0.2" already present on machine
  Normal   Created    2m1s                kubelet            Created container server
  Normal   Started    2m1s                kubelet            Started container server
  Warning  Unhealthy  1s (x12 over 102s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
```
Readeness проба не прошла, так как нет коррекного ответа.

### DaemonSet
Создаём файл [node-exporter-daemonset.yaml](./kubernetes-controllers/node-exporter-daemonset.yaml). В нём мы описываем сервис node-exporter и DaemonSet для его запуска.

DaemonSet должен обеспечить нам запуск 1 экземпляра сервиса на всех worker нодах кластера.

```bash
kubectl apply -f node-exporter-daemonset.yaml

kubectl get po -n kube-system

NAME                                          READY   STATUS    RESTARTS        AGE
...
coredns-64897985d-gnxq5                       1/1     Running  
prometheus-node-exporter-5h87k                1/1     Running   0               60s
prometheus-node-exporter-bb2gd                1/1     Running   0               60s
prometheus-node-exporter-f645q                1/1     Running   0               61s
```
Как видно из вывода node-exporter запущен на всех worker нодах кластера.

Для того чтобы запустить node-exporter на всех нодах кластера, без внесения изменений в настройки узлов нужно воспользоваться механизмом tolerations.  

Каждая control-plane  нода имеет свойство Taints:  node-role.kubernetes.io/master:NoSchedule,  которое запрещает запуск подов на этих узлах. Мы можем изменить это поведение для отдельных подов.

Для этого добавим в спецификацию DaemonSet следующие строки
```yaml
tolerations:
  - key: "node-role.kubernetes.io/master"
    effect: "NoSchedule"
    operator: "Exists"
```
После применения изменённого манифеста результат поменялся на нужный.

```bash
kubectl get po -n kube-system
NAME                                          READY   STATUS    RESTARTS      
...
prometheus-node-exporter-5vspn                1/1     Running   0             55s
prometheus-node-exporter-cpf6m                1/1     Running   0             19s
prometheus-node-exporter-fq227                1/1     Running   0             32s
prometheus-node-exporter-lcfrc                1/1     Running   0             55s
prometheus-node-exporter-lpdnp                1/1     Running   0             55s
prometheus-node-exporter-sqx65                1/1     Running   0             27s
```
node-exporter запущен на всех 6 узлах кластера, как и требовалось в задании.


# Выполнено ДЗ № 3

 - [*] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
### Задача 1
Создан манифест yaml [01-sa-bob.yaml](./kubernetes-security/task01/01-sa-bob.yaml) для создания серисного аккаунта **bob**

Применение манифеста создаёт serviceaccount

```bash
kubectl apply -f 01-sa-bob.yaml 
serviceaccount/bob created
```


Создан манифест yaml [02-role-binding.yaml](./kubernetes-security/task01/02-role-binding.yaml) для привязки серисного аккаунта **bob** к роли **admin** в рамках всего кластера.

Применение этого манифеста создаёт привязку аккаунта к роли

```bash
kubectl apply -f 02-role-binding.yaml 
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
```

Создан манифест yaml [01-sa-dave.yaml](./kubernetes-security/task01/01-sa-dave.yaml) для создания серисного аккаунта **dave**

Применение манифеста

```bash
kubectl apply -f 03-sa-dave.yaml 
serviceaccount/dave created
```

По умолчанию сервис аккаунт не имеет доступа к кластеру.


### Задание 2

Создан манифест yaml [01-namespace.yaml](./kubernetes-security/task02/01-namespace.yaml) для создания неймспейса **prometheus**

Применяем

```bash
kubectl apply -f 01-namespace.yaml 
namespace/prometheus created
```

Создан манифест yaml [02-sa-carol.yaml](./kubernetes-security/task02/02-sa-carol.yaml) для создания серисного аккаунта **carol**

Применяем

```bash
kubectl apply -f 02-sa-carol.yaml 
serviceaccount/carol created
```

Создаём манифест [03-role-pod-reader.yaml](./kubernetes-security/task02/03-role-pod-reader.yaml) для роли, котораяя может выполнять ограниченный набор действий в кластере.

Применяем

```bash
kubectl apply -f 03-role-pod-reader.yaml -n prometheus
role.rbac.authorization.k8s.io/cluster-pod-reader created
```

Создаём манифест [04-sa-set-role.yaml](./kubernetes-security/task02/04-sa-set-role.yaml) для применения этой роли ко всем аккаунтам неймспейса **prometheus**

```bash
kubectl apply -f 04-sa-set-role.yaml -n prometheus
clusterrolebinding.rbac.authorization.k8s.io/cluster-pod-viwer created
```

### Задание 3

Создан манифест yaml [01-namespace.yaml](./kubernetes-security/task03/01-namespace.yaml) для создания неймспейса **dev**

```bash
kubectl apply -f 01-namespace.yaml 
namespace/dev created
```
Сервис аккаунт **jane** [02-sa-jane.yaml](./kubernetes-security/task03/02-sa-jane.yaml)
```bash
kubectl apply -f 02-sa-jane.yaml 
serviceaccount/jane created
```

Привязка **jane** к роли **admin** в неймспейс **dev**

```bash
kubectl apply -f 03-role-jane.yaml 
rolebinding.rbac.authorization.k8s.io/dev-admin-binding created
```

Сервис аккаунт **ken** [04-sa-ken.yaml](./kubernetes-security/task03/04-sa-ken.yaml)

```bash
kubectl apply -f 04-sa-ken.yaml 
serviceaccount/ken created
```

Привязка **ken** к роли **viewer** в неймспейс **dev**

```bash
kubectl apply -f 05-role-ken.yaml 
rolebinding.rbac.authorization.k8s.io/dev-viewer-binding created
```







