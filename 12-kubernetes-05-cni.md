# Домашнее задание к занятию "12.5 Сетевые решения CNI"
> После работы с Flannel появилась необходимость обеспечить безопасность для приложения. Для этого лучше всего подойдет Calico.
> ## Задание 1: установить в кластер CNI плагин Calico
> Для проверки других сетевых решений стоит поставить отличный от Flannel плагин — например, Calico. Требования: 
> * установка производится через ansible/kubespray;
> * после применения следует настроить политику доступа к hello-world извне. Инструкции [kubernetes.io](https://kubernetes.io/docs/concepts/services-networking/network-policies/), [Calico](https://docs.projectcalico.org/about/about-network-policy)

* [inventory](12-05/inventory/mycluster/hosts.yaml) файл
* [playbook](12-05/ansible/hello.yml) для установки hello-node
```
vagrant@vagrant:~/kubespray$ yc compute instance list
+----------------------+-------+---------------+---------+---------------+-------------+
|          ID          | NAME  |    ZONE ID    | STATUS  |  EXTERNAL IP  | INTERNAL IP |
+----------------------+-------+---------------+---------+---------------+-------------+
| fhm9aqh22bodq1k4lojo | cp1   | ru-central1-a | RUNNING | 51.250.64.37  | 10.128.0.16 |
| fhmir86qpco1l473n0ug | node1 | ru-central1-a | RUNNING | 51.250.82.211 | 10.128.0.11 |
+----------------------+-------+---------------+---------+---------------+-------------+
```

```
vagrant@vagrant:~/kubespray$ kubectl get nodes -o wide
NAME    STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
cp1     Ready    control-plane   143m   v1.24.3   10.128.0.16   <none>        Ubuntu 20.04.4 LTS   5.4.0-122-generic   containerd://1.6.6
node1   Ready    <none>          141m   v1.24.3   10.128.0.11   <none>        Ubuntu 20.04.4 LTS   5.4.0-122-generic   containerd://1.6.6
```

```
vagrant@vagrant:~/kubespray$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
hello-node-6d5f754cc9-btkgm   1/1     Running   0          23m   10.233.102.136   node1   <none>           <none>
hello-node-6d5f754cc9-lcg9r   1/1     Running   0          23m   10.233.102.135   node1   <none>           <none>
```

```
vagrant@vagrant:~/kubespray$ kubectl get services -o wide
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE    SELECTOR
hello-node   LoadBalancer   10.233.26.6   <pending>     8080:30524/TCP   132m   app=hello-node
kubernetes   ClusterIP      10.233.0.1    <none>        443/TCP          149m   <none>
```
* [00-default](12-05/network-policy/00-default.yaml) - политика "запрещено, что не разрешено"
* [10-hello-node](12-05/network-policy/10-hello-node.yaml) - политика, разрешающая трафик между подами hello-node

Проверяем внутренний трафик без политик - работает:
```
vagrant@vagrant:~/kubespray$ kubectl exec hello-node-6d5f754cc9-btkgm -- curl -m 1 -s http://10.233.102.135:8080 | grep -e request_uri -e host -e client_address
client_address=10.233.102.136
request_uri=http://10.233.102.135:8080/
host=10.233.102.135:8080
vagrant@vagrant:~/kubespray$ kubectl exec hello-node-6d5f754cc9-lcg9r -- curl -m 1 -s http://10.233.102.136:8080 | grep -e request_uri -e host -e client_address
client_address=10.233.102.135
request_uri=http://10.233.102.136:8080/
host=10.233.102.136:8080
```

Проверяем доступ извне - доступ есть:
```
vagrant@vagrant:~/kubespray$ curl -s -m 1 http://51.250.64.37:30524 | grep -e request_uri -e host -e client_address
client_address=10.233.116.128
request_uri=http://51.250.64.37:8080/
host=51.250.64.37:30524
```

Применяем политики:
```
vagrant@vagrant:~/kubespray$ kubectl apply -f network-policy/00-default.yaml 
networkpolicy.networking.k8s.io/default-deny created
vagrant@vagrant:~/kubespray$ kubectl apply -f network-policy/10-hello-node.yaml 
networkpolicy.networking.k8s.io/internal-accept created
vagrant@vagrant:~/kubespray$ kubectl get networkpolicies
NAME              POD-SELECTOR     AGE
default-deny      <none>           26s
internal-accept   app=hello-node   19s
```
Проверяем трафик между подами - также работает:
```
vagrant@vagrant:~/kubespray$ kubectl exec hello-node-6d5f754cc9-btkgm -- curl -m 1 -s http://10.233.102.135:8080 | grep -e request_uri -e host -e client_address
client_address=10.233.102.136
request_uri=http://10.233.102.135:8080/
host=10.233.102.135:8080
vagrant@vagrant:~/kubespray$ kubectl exec hello-node-6d5f754cc9-lcg9r -- curl -m 1 -s http://10.233.102.136:8080 | grep -e request_uri -e host -e client_address
client_address=10.233.102.135
request_uri=http://10.233.102.136:8080/
host=10.233.102.136:8080
```

Проверяем доступ извне - доступ отсутствует:
```
vagrant@vagrant:~/kubespray$ curl -s -v -m 1 http://51.250.64.37:30524 | grep -e request_uri -e host -e client_address
*   Trying 51.250.64.37:30524...
* TCP_NODELAY set
* Connection timed out after 1002 milliseconds
* Closing connection 0
```

> ## Задание 2: изучить, что запущено по умолчанию
> Самый простой способ — проверить командой calicoctl get <type>. Для проверки стоит получить список нод, ipPool и profile.
> Требования: 
> * установить утилиту calicoctl;
> * получить 3 вышеописанных типа в консоли.

```
vagrant@vagrant:~/kubespray$ calicoctl get node
NAME    
cp1     
node1 

vagrant@vagrant:~/kubespray$ calicoctl get ipPool
NAME           CIDR             SELECTOR   
default-pool   10.233.64.0/18   all()      

vagrant@vagrant:~/kubespray$ calicoctl get profile
NAME                                                 
projectcalico-default-allow                          
kns.default                                          
kns.kube-node-lease                                  
kns.kube-public                                      
kns.kube-system                                      
ksa.default.default                                  
ksa.kube-node-lease.default                          
ksa.kube-public.default                              
ksa.kube-system.attachdetach-controller              
ksa.kube-system.bootstrap-signer                     
ksa.kube-system.calico-node                          
ksa.kube-system.certificate-controller               
ksa.kube-system.clusterrole-aggregation-controller   
ksa.kube-system.coredns                              
ksa.kube-system.cronjob-controller                   
ksa.kube-system.daemon-set-controller                
ksa.kube-system.default                              
ksa.kube-system.deployment-controller                
ksa.kube-system.disruption-controller                
ksa.kube-system.dns-autoscaler                       
ksa.kube-system.endpoint-controller                  
ksa.kube-system.endpointslice-controller             
ksa.kube-system.endpointslicemirroring-controller    
ksa.kube-system.ephemeral-volume-controller          
ksa.kube-system.expand-controller                    
ksa.kube-system.generic-garbage-collector            
ksa.kube-system.horizontal-pod-autoscaler            
ksa.kube-system.job-controller                       
ksa.kube-system.kube-proxy                           
ksa.kube-system.namespace-controller                 
ksa.kube-system.node-controller                      
ksa.kube-system.nodelocaldns                         
ksa.kube-system.persistent-volume-binder             
ksa.kube-system.pod-garbage-collector                
ksa.kube-system.pv-protection-controller             
ksa.kube-system.pvc-protection-controller            
ksa.kube-system.replicaset-controller                
ksa.kube-system.replication-controller               
ksa.kube-system.resourcequota-controller             
ksa.kube-system.root-ca-cert-publisher               
ksa.kube-system.service-account-controller           
ksa.kube-system.service-controller                   
ksa.kube-system.statefulset-controller               
ksa.kube-system.token-cleaner                        
ksa.kube-system.ttl-after-finished-controller        
ksa.kube-system.ttl-controller 
```
