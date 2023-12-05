# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.
3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.
4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.
2. Создать Deployment приложения _backend_ из образа multitool. 
3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 
4. Продемонстрировать, что приложения видят друг друга с помощью Service.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

Создаем пространство имен под ДЗ:
```
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl apply -f ~/manifests/04_dz_kuber_1.5/01_namespace.yml
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   4d11h
kube-public       Active   4d11h
kube-node-lease   Active   4d11h
default           Active   4d11h
lesson4           Active   4d11h
ingress           Active   4d11h
dz5               Active   29s
```
Поднимаю деплоймент фронтэнда
```
kubectl apply -f ~/manifests/04_dz_kuber_1.5/02_deploy_svc_nginx.yml
deployment.apps/dpl-nginx-dz5 created
service/svc-fe-dz5 created

usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get pods -n dz5 -o wide
NAME                             READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-dz5-5cb659b4f5-6lfnn   1/1     Running   0          3m49s   10.1.62.212   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-hl5k2   1/1     Running   0          3m49s   10.1.62.211   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-t5n5h   1/1     Running   0          3m49s   10.1.62.213   microk8s-03   <none>           <none>

usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get deployments -n dz5 -o wide
NAME            READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
dpl-nginx-dz5   3/3     3            3           4m19s   nginx        nginx:1.19.2   app=fe-dz5

usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get svc -n dz5 -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
svc-fe-dz5   ClusterIP   10.152.183.69   <none>        80/TCP    5m14s   app=fe-dz5

usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get ep -n dz5 -o wide
NAME         ENDPOINTS                                      AGE
svc-fe-dz5   10.1.62.211:80,10.1.62.212:80,10.1.62.213:80   5m22s

```

Поднимаю тестовый ПОД:
```
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl apply -f ~/manifests/04_dz_kuber_1.5/03_pod_multitool.yml
pod/pod-multitool-dz5 created

usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get pods -n dz5 -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-dz5-5cb659b4f5-6lfnn   1/1     Running   0          19m   10.1.62.212   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-hl5k2   1/1     Running   0          19m   10.1.62.211   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-t5n5h   1/1     Running   0          19m   10.1.62.213   microk8s-03   <none>           <none>
pod-multitool-dz5                1/1     Running   0          36s   10.1.62.215   microk8s-03   <none>           <none>
```

Тестим:
```
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl exec -n dz5 pod-multitool-dz5 -- curl svc-fe-dz5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   185k      0 --:--:-- --:--:-- --:--:--  298k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Теперь бэкенд:
```
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl apply -f ~/manifests/04_dz_kuber_1.5/04_deploy_svc_multitool.yml
deployment.apps/dpl-multitool-dz5 created
service/svc-be-dz5 created
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get pods -n dz5 -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-dz5-5cb659b4f5-6lfnn       1/1     Running   0          29m   10.1.62.212   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-hl5k2       1/1     Running   0          29m   10.1.62.211   microk8s-03   <none>           <none>
dpl-nginx-dz5-5cb659b4f5-t5n5h       1/1     Running   0          29m   10.1.62.213   microk8s-03   <none>           <none>
pod-multitool-dz5                    1/1     Running   0          10m   10.1.62.215   microk8s-03   <none>           <none>
dpl-multitool-dz5-64455fd7b6-qrsb7   1/1     Running   0          15s   10.1.62.216   microk8s-03   <none>           <none>
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get deployments -n dz5 -o wide
NAME                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                    SELECTOR
dpl-nginx-dz5       3/3     3            3           29m   nginx        nginx:1.19.2              app=fe-dz5
dpl-multitool-dz5   1/1     1            1           44s   multitool    wbitt/network-multitool   app=be-dz5
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get svc -n dz5 -o wide
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
svc-fe-dz5   ClusterIP   10.152.183.69    <none>        80/TCP    29m   app=fe-dz5
svc-be-dz5   ClusterIP   10.152.183.129   <none>        80/TCP    57s   app=be-dz5
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl get ep -n dz5 -o wide
NAME         ENDPOINTS                                      AGE
svc-fe-dz5   10.1.62.211:80,10.1.62.212:80,10.1.62.213:80   30m
svc-be-dz5   10.1.62.216:80                                 67s
```

И проверяем доступность с нашего тестового ПОДа:
```
usrcon@cli-k8s-01:~/manifests/04_dz_kuber_1.5$ kubectl exec -n dz5 pod-multitool-dz5 -- curl svc-be-dz5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-multitool-dz5-64455fd7b6-qrsb7 - 10.1.62.216 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   150  100   150    0     0  36240      0 --:--:-- --:--:-- --:--:-- 50000
```

Успех!  

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.
2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.
3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
4. Предоставить манифесты и скриншоты или вывод команды п.2.

------
