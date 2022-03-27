# pavel-strokov_platform
pavel-strokov Platform repository

# Выполнено ДЗ №

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







