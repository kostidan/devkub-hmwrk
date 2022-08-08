# Домашнее задание к занятию "13.1 контейнеры, поды, deployment, statefulset, services, endpoints"
> Настроив кластер, подготовьте приложение к запуску в нём. Приложение стандартное: бекенд, фронтенд, база данных. Его можно найти в папке 13-kubernetes-config.

> ## Задание 1: подготовить тестовый конфиг для запуска приложения
> Для начала следует подготовить запуск приложения в stage окружении с простыми настройками. Требования:
> * под содержит в себе 2 контейнера — фронтенд, бекенд;
> * регулируется с помощью deployment фронтенд и бекенд;
> * база данных — через statefulset.

* [deployment frontend/backend](13-01/manifests/stage/fb.yaml)
* [statefulset database](13-01/manifests/stage/db.yaml)

```
vagrant@vagrant:~/devkub-hmwrk$ kubectl get pods -n stage -o wide
NAME                  READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
db-0                  1/1     Running   0          13m     10.233.102.141   node1   <none>           <none>
fb-5c564549bd-r4jmz   2/2     Running   0          8m28s   10.233.102.142   node1   <none>           <none>

vagrant@vagrant:~/devkub-hmwrk$ kubectl get deployments -n stage
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
fb     1/1     1            1           14m

vagrant@vagrant:~/devkub-hmwrk$ kubectl get statefulset -n stage
NAME   READY   AGE
db     1/1     14m

vagrant@vagrant:~/devkub-hmwrk$ kubectl get svc -n stage
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
db     ClusterIP   10.233.3.205   <none>        5432/TCP   14m
```

> ## Задание 2: подготовить конфиг для production окружения
> Следующим шагом будет запуск приложения в production окружении. Требования сложнее:
> * каждый компонент (база, бекенд, фронтенд) запускаются в своем поде, регулируются отдельными deployment’ами;
> * для связи используются service (у каждого компонента свой);
> * в окружении фронта прописан адрес сервиса бекенда;
> * в окружении бекенда прописан адрес сервиса базы данных.

* [deployment frontend](13-01/manifests/prod/front.yaml)
* [deployment backend](13-01/manifests/prod/back.yaml)
* [statefulset database](13-01/manifests/prod/db.yaml)

```
vagrant@vagrant:~/devkub-hmwrk$ kubectl get pods -n prod -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
backend-app-5599f6c6c-6kz4b     1/1     Running   0          32s   10.233.102.144   node1   <none>           <none>
db-0                            1/1     Running   0          26s   10.233.102.145   node1   <none>           <none>
frontend-app-54d7b9db8b-vgr4r   1/1     Running   0          38s   10.233.102.143   node1   <none>           <none>

vagrant@vagrant:~/devkub-hmwrk$ kubectl get deployments -n prod
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
backend-app    1/1     1            1           70s
frontend-app   1/1     1            1           76s

vagrant@vagrant:~/devkub-hmwrk$ kubectl get statefulset -n prod
NAME   READY   AGE
db     1/1     92s

vagrant@vagrant:~/devkub-hmwrk$ kubectl get svc -n prod
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend-app    ClusterIP   10.233.32.93    <none>        9000/TCP   110s
db             ClusterIP   10.233.48.66    <none>        5432/TCP   104s
frontend-app   ClusterIP   10.233.16.103   <none>        80/TCP     115s
```
