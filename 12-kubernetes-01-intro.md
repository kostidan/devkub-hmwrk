# Домашнее задание к занятию "12.1 Компоненты Kubernetes"

> Вы DevOps инженер в крупной компании с большим парком сервисов. Ваша задача — разворачивать эти продукты в корпоративном кластере. 

## Задача 1: Установить Minikube

> Для экспериментов и валидации ваших решений вам нужно подготовить тестовую среду для работы с Kubernetes. Оптимальное решение — развернуть на рабочей машине Minikube.
>
> <details><summary>Инструкция по установке</summary>
>
> ### Как поставить на AWS:
> - создать EC2 виртуальную машину (Ubuntu Server 20.04 LTS (HVM), SSD Volume Type) с типом **t3.small**. Для работы потребуется настроить Security Group для доступа по ssh. Не забудьте указать keypair, он потребуется для подключения.
> - подключитесь к серверу по ssh (ssh ubuntu@<ipv4_public_ip> -i <keypair>.pem)
> - установите миникуб и докер следующими командами:
>   - curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
>   - chmod +x ./kubectl
>   - sudo mv ./kubectl /usr/local/bin/kubectl
>   - sudo apt-get update && sudo apt-get install docker.io conntrack -y
>   - curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
> - проверить версию можно командой minikube version
> - переключаемся на root и запускаем миникуб: minikube start --vm-driver=none
> - после запуска стоит проверить статус: minikube status
> - запущенные служебные компоненты можно увидеть командой: kubectl get pods --namespace=kube-system
> 
> ### Для сброса кластера стоит удалить кластер и создать заново:
> - minikube delete
> - minikube start --vm-driver=none
> 
> Возможно, для повторного запуска потребуется выполнить команду: sudo sysctl fs.protected_regular=0
> 
> Инструкция по установке Minikube - [ссылка](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/)
> 
> **Важно**: t3.small не входит во free tier, следите за бюджетом аккаунта и удаляйте виртуалку.
>
> </details>

Установка в `yandex.cloud`

```
root@minikube:~# minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

root@minikube:~# kubectl get pods --namespace=kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-64897985d-wtzzt            1/1     Running   0          14s
etcd-minikube                      1/1     Running   0          26s
kube-apiserver-minikube            1/1     Running   0          28s
kube-controller-manager-minikube   1/1     Running   0          26s
kube-proxy-bxxr7                   1/1     Running   0          15s
kube-scheduler-minikube            1/1     Running   0          26s
storage-provisioner                1/1     Running   0          23s
```

## Задача 2: Запуск Hello World

> После установки Minikube требуется его проверить. Для этого подойдет стандартное приложение hello world. А для доступа к нему потребуется ingress.
> 
> - развернуть через Minikube тестовое приложение по [туториалу](https://kubernetes.io/ru/docs/tutorials/hello-minikube/#%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BB%D0%B0%D1%81%D1%82%D0%B5%D1%80%D0%B0-minikube)
> - установить аддоны ingress и dashboard

### Развернуть через Minikube тестовое приложение

Сервисы
```
root@minikube:~/hello-node# kubectl get deployment | grep hello
hello-node   1/1     1            1           17m
```
Деплоймент
```
root@minikube:~/hello-node# kubectl get services | grep hello
hello-node   LoadBalancer   10.99.65.186   <pending>     8080:32177/TCP   16m
```
Под
```
root@minikube:~/hello-node# kubectl get pods | grep hello
hello-node-6b89d599b9-qx5xz   1/1     Running   0          20m
```
Сервисы minikube
```
root@minikube:~/hello-node# minikube service list | grep hello
| default              | hello-node                         |         8080 | http://10.129.0.33:32177 |
```
### Установить аддоны ingress и dashboard

```
root@minikube:~/hello-node# minikube addons list | grep enabled
| dashboard                   | minikube | enabled ✅   | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| ingress                     | minikube | enabled ✅   | unknown (third-party)          |
| storage-provisioner         | minikube | enabled ✅   | google                         |                       |
```

## Задача 3: Установить kubectl

> Подготовить рабочую машину для управления корпоративным кластером. Установить клиентское приложение kubectl.
> - подключиться к minikube 
> - проверить работу приложения из задания 2, запустив port-forward до кластера


* Подключение к `minikube`
  ```
  root@vagrant:~# kubectl version --short
  Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
  Client Version: v1.24.2
  Kustomize Version: v4.5.4
  Server Version: v1.23.3
  ```
* Проверка `port-forward`
  ```
  root@vagrant:~# kubectl port-forward service/hello-node 8080:8080
  Forwarding from 127.0.0.1:8080 -> 8080
  Forwarding from [::1]:8080 -> 8080
  ^[|Handling connection for 8080
  ```

* Вывод `curl`
  ```
  root@vagrant:~# curl localhost:8080
  CLIENT VALUES:
  client_address=127.0.0.1
  command=GET
  real path=/
  query=nil
  request_version=1.1
  request_uri=http://localhost:8080/

  SERVER VALUES:
  server_version=nginx: 1.10.0 - lua: 10001

  HEADERS RECEIVED:
  accept=*/*
  host=localhost:8080
  user-agent=curl/7.74.0
  BODY:
  -no body in request-0
  ```
