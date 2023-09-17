# elcovstas_platform
elcovstas Platform repository

# Домашняя работа 1. Настройка локального окружения. Запуск первого контейнера. Работа с kubectl

1. Установил minicube и kubectl

```
selcov@ubuntu:~/elcovstas_platform$ kubectl version --short
Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
Client Version: v1.25.2
Kustomize Version: v4.5.7
Server Version: v1.25.0
```

```
selcov@ubuntu:~/elcovstas_platform$ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-565d847f94-4fwn8           1/1     Running   0          7m56s
etcd-minikube                      1/1     Running   1          7m56s
kube-apiserver-minikube            1/1     Running   1          7m56s
kube-controller-manager-minikube   1/1     Running   1          7m56s
kube-proxy-nhwhq                   1/1     Running   0          7m54s
kube-scheduler-minikube            1/1     Running   1          7m56s
metrics-server-769cd898cd-kxxdd    1/1     Running   0          7m56s
```

Вопрос 1.
Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR

Причина:

Pod core-dns восстанавливается через Deployment.

```
selcov@ubuntu:~/elcovstas_platform$ kubectl get deployment -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
coredns          1/1     1            1           36m
```

Остальные компоненты - это static pod

В документации сказано, что static pod управляются непосредственно демоном kubelet на определенном узле, без наблюдения за ними сервера API. В отличие от модулей, которые управляются плоскостью управления (например, развертыванием); вместо этого kubelet отслеживает каждый статический модуль (и перезапускает его в случае сбоя). Ссылка на документацию https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

Static pod описываются по пути /etc/kubernetes/manifests

```
$ pwd
/etc/kubernetes/manifests
$ ls -l
total 16
-rw------- 1 root root 2482 Sep 10 08:08 etcd.yaml
-rw------- 1 root root 3642 Sep 10 08:08 kube-apiserver.yaml
-rw------- 1 root root 2951 Sep 10 08:08 kube-controller-manager.yaml
-rw------- 1 root root 1441 Sep 10 08:08 kube-scheduler.yaml
```

Создан Dockerfile и образ k8s-intro-web закинул в Dockerhub.

В основе образа выбрал nginx-unprivileged:1.25-alpine и использовал user c id 1001
В html файле отображается вывод "Hello World!"

Cкачать можете образ по команде:

```
docker push elcovstas/k8s-intro-web:0.0.1
```
Написал манифест, который запускает образ elcovstas/k8s-intro-web:0.0.1 и добавил init контейнер и директории.

Проверка работы:

```
selcov@ubuntu:~$ curl -lvvvv http://127.0.0.1:8000/homework.html
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8000 (#0)
> GET /homework.html HTTP/1.1
> Host: 127.0.0.1:8000
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.25.2
< Date: Sun, 10 Sep 2023 10:24:39 GMT
< Content-Type: text/html
< Content-Length: 133
< Last-Modified: Sun, 10 Sep 2023 08:59:58 GMT
< Connection: keep-alive
< ETag: "64fd858e-85"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
        <title>kubernetes-intro</title>
</head>
<body>
        <div>
                <p>"Hello World!"</p>
        </div>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact

selcov@ubuntu:~$ curl -lvvvv http://127.0.0.1:8000
* Rebuilt URL to: http://127.0.0.1:8000/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8000 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1:8000
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.25.2
< Date: Sun, 10 Sep 2023 10:25:44 GMT
< Content-Type: text/html
< Content-Length: 83486
< Last-Modified: Sun, 10 Sep 2023 10:25:28 GMT
< Connection: keep-alive
< ETag: "64fd9998-1461e"
< Accept-Ranges: bytes
<
<html>
<head/>
<body>
<!-- IMAGE BEGINS HERE -->
<font size="-3">
<pre><font color=white>0111010011111011110010000111011000001110000110010011101000001100101011110010100111010001111101001011000001110110101110111001000110</font><br><font color=white>10010000001000111011000101110101110010110101111110011011011001001111111011011010010011110101111001111010100100110110100101001
**********
export KUBERNETES_PORT_443_TCP_PROTO='tcp'
export KUBERNETES_SERVICE_HOST='10.96.0.1'
export KUBERNETES_SERVICE_PORT='443'
export KUBERNETES_SERVICE_PORT_HTTPS='443'
export PATH='/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
export PWD='/'
export SHLVL='2'</pre>
<h3>Memory info</h3>
<pre>              total        used        free      shared  buff/cache   available
Mem:           2115         717         105         722        1293         524
Swap:             0           0           0</pre>
<h3>DNS resolvers info</h3>
<pre>nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5</pre>
<h3>Static hosts info</h3>
<pre># Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
172.17.0.2      web</pre>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact
```

Выполнил задачу с *
Проблема в запуске контейнера Frontend из-за необъявленных переменных. Проблему можно выяснить с помощью анализа логов контейнера.

Создал манифест frontend-pod-healthy.yaml с исправленной ошибкой, контейнер запустился корректно.

```
selcov@ubuntu:~/elcovstas_platform/kubernetes-intro$ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
frontend   1/1     Running   0          15m
web        1/1     Running   0          35m
```

Выполнены все задания в домашней работе 1.


# Домашняя работа 2. Kubernetes controllers. ReplicaSet, Deployment, DaemonSet

Работа проводилась в установленном кластере с помощью RKE v1. Всего 6 нод в кластере, 3 master ноды и 3 worker ноды.

По запуску frontend-replicaset.yaml был не добавлены опции selector

```
selector:
    matchLabels:
      app: frontend
```

Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?

ReplicaSet не удаляет существующие pod.

Выдержка из документации
"To update Pods to a new spec in a controlled way, use a Deployment, as ReplicaSets do not support a rolling update directly."

Найдите способ модернизировать свой DaemonSet таким образом, чтобы Node Exporter был развернут как на master, так и на worker нодах (конфигурацию самих нод изменять нельзя);

В мастер нодах, есть такие tains:

```
root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-controllers# kubectl get nodes -o json | jq '.items[].spec.taints'
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/controlplane",
    "value": "true"
  },
  {
    "effect": "NoExecute",
    "key": "node-role.kubernetes.io/etcd",
    "value": "true"
  }
]
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/controlplane",
    "value": "true"
  },
  {
    "effect": "NoExecute",
    "key": "node-role.kubernetes.io/etcd",
    "value": "true"
  }
]
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/controlplane",
    "value": "true"
  },
  {
    "effect": "NoExecute",
    "key": "node-role.kubernetes.io/etcd",
    "value": "true"
  }
]

```

В манифесте можно использовать tolerations.

Запись tolerations в Kubernetes (k8s) описывает толерантность пода к определенным условиям планирования. Он позволяет запускать поды на узлах с установленными точными толерансами, даже если эти узлы находятся в условиях, отказывающих планировщику размещать на них поды.

В данном случае, запись tolerations задает два условия толерантности для пода:

1. effect: NoSchedule и operator: Exists указывают, что под будет толерантен к состоянию узла с запретом на размещение новых подов (NoSchedule). Это позволит планировщику Kubernetes размещать этот под на узле, даже если на этом узле установлено условие NoSchedule.

2. effect: NoExecute и operator: Exists указывают, что под будет толерантен к состоянию узла с запретом на исполнение старых или нежелательных подов (NoExecute). Это позволит Kubernetes оставлять этот под на узле, даже если этот узел имеет состояние NoExecute.

Такие записи tolerations полезны, когда вы хотите запустить поды на узлах, которые находятся во временных состояниях запрета размещения или исполнения, например, из-за отказа ноды или планового обслуживания.



# Домашняя работа 3. Сетевое взаимодействие Pod, сервисы

Вопрос:

Почему следующая конфигурация валидна, но не имеет смысла?

```
livenessProbe:
exec:
command:
- 'sh'
- '-c'
- 'ps aux | grep my_web_server_process'
```

Потому что запущенные процессы не гарантируют работоспособность web сервиса. Web сервер может работать на другом порту или отдавать не то.
Такая проверка имеет смысл, если у нас нет возможности запросить у приложения его статус.

Основное задание сделано по методичке.
Задания со звездочкой:

1. Сделайте сервис LoadBalancer , который откроет доступ к CoreDNS снаружи кластера (позволит получать записи через внешний IP).

Написал манифест dns-service.yaml в папке coredns
Логика такая:
Создать 2 сервиса с с заданым значением metallb.universe.tf/allow-shared-ip

2. Добавьте доступ к kubernetes-dashboard через наш Ingress-прокси:

Написал ingress правило для доступа к kubernetes-dashboard

```
</body></html>root@knd-test-kub-m1:~/elcovstas_platform# curl  http://knd-test-kub-s2.fors.ru/dashboard/
<!--
Copyright 2017 The Kubernetes Authors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
--><!DOCTYPE html><html lang="en" dir="ltr"><head>
  <meta charset="utf-8">
  <title>Kubernetes Dashboard</title>
  <link rel="icon" type="image/png" href="assets/images/kubernetes-logo.png">
  <meta name="viewport" content="width=device-width">
<style>html,body{height:100%;margin:0}*::-webkit-scrollbar{background:transparent;height:8px;width:8px}</style><link rel="stylesheet" href="styles.243e6d874431c8e8.css" media="print" onload="this.media='all'"><noscript><link rel="stylesheet" href="styles.243e6d874431c8e8.css"></noscript></head>

<body>
  <kd-root></kd-root>
<script src="runtime.134ad7745384bed8.js" type="module"></script><script src="polyfills.5c84b93f78682d4f.js" type="module"></script><script src="scripts.2c4f58d7c579cacb.js" defer></script><script src="en.main.3550e3edca7d0ed8.js" type="module"></script>
```

3. Canary для Ingress

Для канаречного релиза необходимо создать копию манифестов для разворачивания новой версии приложения и в инресс добавить несколько антонаций:

nginx.ingress.kubernetes.io/canary: "true"
nginx.ingress.kubernetes.io/canary-by-header: "canary"
nginx.ingress.kubernetes.io/canary-by-weight: "50"