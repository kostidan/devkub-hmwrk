# Домашнее задание к занятию "12.2 Команды для работы с Kubernetes"

> Кластер — это сложная система, с которой крайне редко работает один человек. Квалифицированный devops умеет наладить работу всей команды, занимающейся каким-либо сервисом.
> После знакомства с кластером вас попросили выдать доступ нескольким разработчикам. Помимо этого требуется служебный аккаунт для просмотра логов.

## Задание 1: Запуск пода из образа в деплойменте

> Для начала следует разобраться с прямым запуском приложений из консоли. Такой подход поможет быстро развернуть инструменты отладки в кластере. Требуется запустить деплоймент на основе образа из hello world уже через deployment. Сразу стоит запустить 2 копии приложения (replicas=2). 
> 
> Требования:
>  * пример из hello world запущен в качестве deployment
>  * количество реплик в deployment установлено в 2
>  * наличие deployment можно проверить командой kubectl get deployment
>  * наличие подов можно проверить командой kubectl get pods

* `количество реплик в deployment установлено в 2`
    ```
    root@minikube:~# kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4 -r 2
    deployment.apps/hello-node created
    ```
* `kubectl get deployment`
    ```
    root@minikube:~# kubectl get deployments
    NAME         READY   UP-TO-DATE   AVAILABLE   AGE
    hello-node   2/2     2            2           3s
    ```
* `kubectl get pods`
    ```
    root@minikube:~# kubectl get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    hello-node-6d5f754cc9-4mz22   1/1     Running   0          9s
    hello-node-6d5f754cc9-d5ltc   1/1     Running   0          9s
    ```

## Задание 2: Просмотр логов для разработки

> Разработчикам крайне важно получать обратную связь от штатно работающего приложения и, еще важнее, об ошибках в его работе. 
> Требуется создать пользователя и выдать ему доступ на чтение конфигурации и логов подов в app-namespace.
> 
> Требования: 
>  * создан новый токен доступа для пользователя
>  * пользователь прописан в локальный конфиг (~/.kube/config, блок users)
>  * пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>)

* `/.kube/config`
    ```
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /root/.minikube/ca.crt
        extensions:
        - extension:
            last-update: Sun, 03 Jul 2022 16:30:39 UTC
            provider: minikube.sigs.k8s.io
            version: v1.26.0
        name: cluster_info
        server: https://10.129.0.14:8443
    name: minikube
    contexts:
    - context:
        cluster: minikube
        namespace: app-namespace
        user: user
    name: app-namespace-dev
    - context:
        cluster: minikube
        extensions:
        - extension:
            last-update: Sun, 03 Jul 2022 16:30:39 UTC
            provider: minikube.sigs.k8s.io
            version: v1.26.0
        name: context_info
        namespace: default
        user: minikube
    name: minikube
    current-context: app-namespace-dev
    kind: Config
    preferences: {}
    users:
    - name: user
    user:
        client-certificate: /root/user/user.crt
        client-key: /root/user/user.key
    - name: minikube
    user:
        client-certificate: /root/.minikube/profiles/minikube/client.crt
        client-key: /root/.minikube/profiles/minikube/client.key
    ```
* `role.yml`
    ```
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
    namespace: app-namespace
    name: role-dev
    rules:
    - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list"]
    ```
* `rolebinding.yml`
    ```
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
    name: rolebinding-dev
    namespace: app-namespace
    subjects:
    - kind: User
    name: user
    apiGroup: rbac.authorization.k8s.io
    roleRef:
    kind: Role
    name: role-dev
    apiGroup: rbac.authorization.k8s.io
    ```
* `Переключение контекста и проверка прав пользователя`
    ```
    root@minikube:~/user# kubectl config use-context app-namespace-dev
    Switched to context "app-namespace-dev".
    root@minikube:~/user# kubectl get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    hello-node-6d5f754cc9-v98rs   1/1     Running   0          17m
    root@minikube:~/user# kubectl describe pods | head -n 3
    Name:         hello-node-6d5f754cc9-v98rs
    Namespace:    app-namespace
    Priority:     0
    root@minikube:~/user# kubectl logs pods/hello-node-6d5f754cc9-v98rs
    root@minikube:~/user# kubectl delete pod hello-node-6d5f754cc9-v98rs
    Error from server (Forbidden): pods "hello-node-6d5f754cc9-v98rs" is forbidden: User "user" cannot delete resource "pods" in API group "" in the namespace "app-namespace"
    root@minikube:~/user# kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
    error: failed to create deployment: deployments.apps is forbidden: User "user" cannot create resource "deployments" in API group "apps" in the namespace "app-namespace"
    ```


## Задание 3: Изменение количества реплик 

> Поработав с приложением, вы получили запрос на увеличение количества реплик приложения для нагрузки. Необходимо изменить запущенный deployment, увеличив количество реплик до 5. Посмотрите статус запущенных подов после увеличения реплик. 
> 
> Требования:
>  * в deployment из задания 1 изменено количество реплик на 5
>  * проверить что все поды перешли в статус running (kubectl get pods)

* `Увеличим количество реплик до 5`
    ```
    root@minikube:~/user# kubectl scale deployment hello-node --replicas=5
    deployment.apps/hello-node scaled
    root@minikube:~/user# kubectl get pods
    NAME                          READY   STATUS    RESTARTS   AGE
    hello-node-6d5f754cc9-4mz22   1/1     Running   0          93m
    hello-node-6d5f754cc9-d5ltc   1/1     Running   0          93m
    hello-node-6d5f754cc9-j9vwx   1/1     Running   0          116s
    hello-node-6d5f754cc9-qfmrn   1/1     Running   0          116s
    hello-node-6d5f754cc9-x7dg4   1/1     Running   0          116s
    ```