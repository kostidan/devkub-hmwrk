# Домашнее задание к занятию "12.4 Развертывание кластера на собственных серверах, лекция 2"
> Новые проекты пошли стабильным потоком. Каждый проект требует себе несколько кластеров: под тесты и продуктив. Делать все руками — не вариант, поэтому стоит автоматизировать подготовку новых кластеров.

> ## Задание 1: Подготовить инвентарь kubespray
> Новые тестовые кластеры требуют типичных простых настроек. Нужно подготовить инвентарь и проверить его работу. Требования к инвентарю:
> * подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды;
> * в качестве CRI — containerd;
> * запуск etcd производить на мастере.

* В k8s-cluster.yml включил параметр supplementary_addresses_in_ssl_keys: [51.250.10.214], чтобы прописать сертификат для внешнего ip мастер ноды для доступа к кластеру с локальной машины
* [inventory](12-03-part-2/hosts.yaml) файл

Ноды:

```
yc-user@cp1:~$ kubectl get nodes
NAME    STATUS   ROLES           AGE    VERSION
cp1     Ready    control-plane   27m    v1.24.3
node1   Ready    <none>          25m    v1.24.3
node2   Ready    <none>          6m9s   v1.24.3
node3   Ready    <none>          6m8s   v1.24.3
node4   Ready    <none>          6m9s   v1.24.3
```

Поды:

```
yc-user@cp1:~$ kubectl get pod -o wide -A
NAMESPACE     NAME                              READY   STATUS    RESTARTS      AGE     IP               NODE    NOMINATED NODE   READINESS GATES
kube-system   calico-node-75nfz                 1/1     Running   0             26m     10.128.0.3       node1   <none>           <none>
kube-system   calico-node-9mbwt                 1/1     Running   0             7m26s   10.128.0.36      node4   <none>           <none>
kube-system   calico-node-c7l6r                 1/1     Running   0             7m26s   10.128.0.5       node2   <none>           <none>
kube-system   calico-node-vdprb                 1/1     Running   0             26m     10.128.0.21      cp1     <none>           <none>
kube-system   calico-node-zslsr                 1/1     Running   1             7m25s   10.128.0.6       node3   <none>           <none>
kube-system   coredns-74d6c5659f-bdjj8          1/1     Running   0             23m     10.233.102.129   node1   <none>           <none>
kube-system   coredns-74d6c5659f-txx82          1/1     Running   0             24m     10.233.116.129   cp1     <none>           <none>
kube-system   dns-autoscaler-59b8867c86-qfx87   1/1     Running   0             23m     10.233.116.130   cp1     <none>           <none>
kube-system   kube-apiserver-cp1                1/1     Running   1             28m     10.128.0.21      cp1     <none>           <none>
kube-system   kube-controller-manager-cp1       1/1     Running   2 (23m ago)   28m     10.128.0.21      cp1     <none>           <none>
kube-system   kube-proxy-d5mck                  1/1     Running   0             7m22s   10.128.0.21      cp1     <none>           <none>
kube-system   kube-proxy-jm57k                  1/1     Running   0             7m22s   10.128.0.6       node3   <none>           <none>
kube-system   kube-proxy-l2xsh                  1/1     Running   0             7m22s   10.128.0.3       node1   <none>           <none>
kube-system   kube-proxy-lqng5                  1/1     Running   0             7m22s   10.128.0.36      node4   <none>           <none>
kube-system   kube-proxy-zgppl                  1/1     Running   0             7m22s   10.128.0.5       node2   <none>           <none>
kube-system   kube-scheduler-cp1                1/1     Running   2 (23m ago)   28m     10.128.0.21      cp1     <none>           <none>
kube-system   nginx-proxy-node1                 1/1     Running   0             25m     10.128.0.3       node1   <none>           <none>
kube-system   nginx-proxy-node2                 1/1     Running   0             6m15s   10.128.0.5       node2   <none>           <none>
kube-system   nginx-proxy-node3                 1/1     Running   0             6m26s   10.128.0.6       node3   <none>           <none>
kube-system   nginx-proxy-node4                 1/1     Running   0             6m6s    10.128.0.36      node4   <none>           <none>
kube-system   nodelocaldns-2nbz7                1/1     Running   0             7m25s   10.128.0.5       node2   <none>           <none>
kube-system   nodelocaldns-gfp49                1/1     Running   0             23m     10.128.0.3       node1   <none>           <none>
kube-system   nodelocaldns-lt27d                1/1     Running   0             7m26s   10.128.0.36      node4   <none>           <none>
kube-system   nodelocaldns-pltbs                1/1     Running   0             7m25s   10.128.0.6       node3   <none>           <none>
kube-system   nodelocaldns-ztcg9                1/1     Running   0             23m     10.128.0.21      cp1     <none>           <none>
```

containerd в CRI

```
yc-user@cp1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
f6dcd00ca7962       2ae1ba6417cbc       8 minutes ago       Running             kube-proxy                0                   4d6ca3c2daf9c       kube-proxy-d5mck
078758fbd9e00       586c112956dfc       23 minutes ago      Running             kube-controller-manager   2                   fa7fa920ae862       kube-controller-manager-cp1
334539e74271a       3a5aa3a515f5d       23 minutes ago      Running             kube-scheduler            2                   2634aa2d04633       kube-scheduler-cp1
e0d3c6d9992b7       d521dd763e2e3       23 minutes ago      Running             kube-apiserver            1                   4c2b89a482441       kube-apiserver-cp1
28b7fde65390a       5bae806f8f123       24 minutes ago      Running             node-cache                0                   942b082ca7ee3       nodelocaldns-ztcg9
15297e9f4b483       1e7da779960fc       24 minutes ago      Running             autoscaler                0                   1a803bc60b7bf       dns-autoscaler-59b8867c86-qfx87
b788b803c522d       a4ca41631cc7a       24 minutes ago      Running             coredns                   0                   4e0ebc591b070       coredns-74d6c5659f-txx82
9500cb8b2c5e8       5f5175f39b19e       25 minutes ago      Running             calico-node               0                   bde686e126ac9       calico-node-vdprb
```

