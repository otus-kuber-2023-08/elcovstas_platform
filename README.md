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


# Домашняя работа 4. Security

1. Выполнен Task 1
2. Выполнен Task 2
3. Выполнен Task 3

# Домашняя работа 5. Volumes, Storages, StatefulSet

Создал StatefulSet minio
Установил local-path-provisioner для Rancher https://github.com/rancher/local-path-provisioner
Создал minio-ingress и minio-secrets


# Домашняя работа 6. Шаблонизация манифестов Kubernetes

1) создал кластер в Яндекс cloud
2) C помощью Helm установил ingress-nginx  https://cloud.yandex.ru/docs/managed-kubernetes/tutorials/ingress-cert-manager
3) C помощью Helm установил cert manager и создал ресурсы ClusterIssuer 
4) Установил chartmuseum с помощью helm с созданным ingress правилом с автоматической генерацией Let's Encrypt сертификата:

```
# Установка чарта

helm upgrade --install chartmuseum chartmuseum/chartmuseum --version 3.1.0 --wait --namespace=chartmuseum --create-namespace  -f chartmuseum/values.yaml

# Содержимое ingress в values 

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
    cert-manager.io/acme-challenge-type: http01
  hosts:
    - name: chartmuseum.51.250.87.87.nip.io
      path: /
      tls: true
      tlsSecret: chartmuseum.51.250.87.87.nip.io

# Проверка генерации сертификата

selcov@ubuntu:~/k8s/elcovstas_platform$ kubectl get certificate -n chartmuseum -o yaml
apiVersion: v1
items:
- apiVersion: cert-manager.io/v1
  kind: Certificate
  metadata:
    creationTimestamp: "2023-10-08T11:16:06Z"
    generation: 1
    labels:
      app.kubernetes.io/instance: chartmuseum
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: chartmuseum
      app.kubernetes.io/version: 0.13.1
      helm.sh/chart: chartmuseum-3.1.0
    name: chartmuseum.51.250.87.87.nip.io
    namespace: chartmuseum
    ownerReferences:
    - apiVersion: networking.k8s.io/v1
      blockOwnerDeletion: true
      controller: true
      kind: Ingress
      name: chartmuseum
      uid: 2d8bef37-6fdb-42fb-af2b-d77817bea831
    resourceVersion: "45055"
    uid: 5fc2ea80-3913-4b12-9e03-6e2f85b36322
  spec:
    dnsNames:
    - chartmuseum.51.250.87.87.nip.io
    issuerRef:
      group: cert-manager.io
      kind: ClusterIssuer
      name: letsencrypt-staging
    secretName: chartmuseum.51.250.87.87.nip.io
    usages:
    - digital signature
    - key encipherment
  status:
    conditions:
    - lastTransitionTime: "2023-10-08T11:16:32Z"
      message: Certificate is up to date and has not expired
      observedGeneration: 1
      reason: Ready
      status: "True"
      type: Ready
    notAfter: "2024-01-06T10:16:27Z"
    notBefore: "2023-10-08T10:16:28Z"
    renewalTime: "2023-12-07T10:16:27Z"
    revision: 1
kind: List
metadata:
  resourceVersion: ""

```

5) Задание со * "Научитесь работать с chartmuseum"

chartmuseum должен быть запушен с переменной DISABLE_API: false 

Пулим redmine из публичного репотизория
```
helm pull stable/redmine --version 14.1.1
```

Добавляем репотизорий https://chartmuseum.51.250.87.87.nip.io/

```
helm repo add my-chartmuseum https://chartmuseum.51.250.87.87.nip.io/ --insecure-skip-tls-verify
```

Устанавливаем плагин

```
helm plugin install https://github.com/chartmuseum/helm-push
```

Пушим

```
helm cm-push --insecure redmine-14.1.10.tgz my-chartmuseum
```

Проверяем 

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-templating/chartmuseum$ curl -vvvk https://chartmuseum.51.250.87.87.nip.io/api/charts
*   Trying 51.250.87.87...
* TCP_NODELAY set
* Connected to chartmuseum.51.250.87.87.nip.io (51.250.87.87) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
********
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
< HTTP/2 200
< date: Sun, 08 Oct 2023 12:36:07 GMT
< content-type: application/json; charset=utf-8
< content-length: 937
< x-request-id: e2f269c45179cd8cead7853cd2d9d0d3
< strict-transport-security: max-age=15724800; includeSubDomains
<
{"redmine":[{"name":"redmine","home":"http://www.redmine.org/","sources":["https://github.com/bitnami/bitnami-docker-redmine"],"version":"14.1.10","description":"A flexible project management web application.","keywords":["redmine","project management","www","http","web","application","ruby","rails"],"maintainers":[{"name":"Bitnami","email":"containers@bitnami.com"}],"icon":"https://bitnami.com/assets/stacks/redmine/img/redmine-stack-220x234.png","apiVersion":"v1","appVersion":"4.1.0","dependencies":[{"name":"mariadb","version":"7.x.x","repository":"https://kubernetes-charts.storage.googleapis.com/","condition":"mariadb.enabled"},{"name":"postgresql","version":"8.x.x","repository":"https://kubernetes-charts.storage.googleapis.com/","condition":"postgresql.enabled"}],"urls":["charts/redmine-14.1.10.tgz"],"created":"2023-10-08T12:26:19.733038054Z","digest":"499a4460cef10d1f2c11a1f23c34500ba7f68349be182e9e772eee22280a0e40"}]}
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* Connection #0 to host chartmuseum.51.250.87.87.nip.io left intact
```
6) Выполнено дополнительное задание с установкой Harbor

```
helm install harbor harbor/harbor --namespace harbor -f values.yaml --create-namespace
```

7) Выполнено задание с * "Написание Helmfile"

8) C помощью Helm установил hipster-shop, параматезировал Helm Chart
9) Вынес Frontend и Redis в отдельный Chart, также подключил зависимости в hipster-shop. Параматезировал Helm Chart
10) Настроил зашифрованные Sercets в Frontend

# Домашняя работа 7. Операторы, CustomResourceDefifinition

В процессе сделано:
Изучены кастомные ресурсы и создан оператор для управления CRD.

вывод команд:

```
$ kubectl get jobs.batch
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           2s         6m3s
restore-mysql-instance-job   1/1           93s        4m40s
Вывод

kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```

# Домашняя работа 8. Мониторинг сервиса в кластере k8s

1) Написал deployment, Config Map и service nginx
2) Написал deployment, Config Map и service nginx-exporter
3) Установил prometheus operator c помощью kubectl
4) Написал ServiceMonitor для nginx-exporter
5) Задеплоил prometheus и Grafana
6) Проверил статусы в интерфейсе prometheus

![prometheus](/kubernetes-monitoring/prometheus.PNG)
7) Создал дашбоард для nginx-exporter
![prometheus](/kubernetes-monitoring/grafana.PNG)

# Домашняя работа 9. Логирование

1) Установил EFK стэк
2) Установил Prometheus, Grafana, Loki
3) Настроил мониторинг ElasticSearch
4) Настроил вывод логов nginx-ingress и остальных контейнеров в EFK и Grafana.
5) Пробовал рисовать Dashboard

# Домашяя работа 11. Hashicorp Vault + K8s

1) Установил consul и Vault с помощью Helm



```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ helm status consul
NAME: consul
LAST DEPLOYED: Sun Dec  3 20:07:53 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Consul!

Your release is named consul.

To learn more about the release, run:

  $ helm status consul --namespace default
  $ helm get all consul --namespace default

Consul on Kubernetes Documentation:
https://www.consul.io/docs/platform/k8s

Consul on Kubernetes CLI Reference:
https://www.consul.io/docs/k8s/k8s-cli

selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ helm status vault
NAME: vault
LAST DEPLOYED: Sun Dec  3 20:15:42 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://developer.hashicorp.com/vault/docs


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault

```

Инициизируем vault

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec -it vault-0  -- vault operator init
Unseal Key 1: 43KZMuFcDmofcgj7UTAd2JfVb7U2v93JIgVLv0CtLvWa
Unseal Key 2: MjwWxDam98t5/xSiHj7V5JX8r6ai82qs4uswUSSm2QMS
Unseal Key 3: 4cKLX53JUOpVgdkG0Q8iBmidp4y5Fk6YfhV6bBMRMA8e
Unseal Key 4: L89PqYKNRSsjG0WPYTg+A4+o0c4F7MtYVEjkNpJRB2lH
Unseal Key 5: PmfS5lnzlt0FZb8hcYys1CTrv8NPtpciYGBc3EYqGIE+

Initial Root Token: hvs.Peff4kxjfGQG1imIUcjPqAvt

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated root key. Without at least 3 keys to
reconstruct the root key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

Распечатываем vault

```
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.15.2
Build Date      2023-11-06T11:33:28Z
Storage Type    consul
Cluster Name    vault-cluster-0f7b639b
Cluster ID      066130cb-db88-289c-30de-fe93ce71e1bc
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2023-12-03T17:22:13.400233148Z
```

Аутентификация в vault

```
Path      Type     Accessor               Description                Version
----      ----     --------               -----------                -------
token/    token    auth_token_f9f94180    token based credentials    n/a

```

Добавим секреты
```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl -n vault exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus
```

Включили авторизацию k8s

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl -n vault exec -it vault-0 -- vault auth list
Path           Type          Accessor                    Description                Version
----           ----          --------                    -----------                -------
kubernetes/    kubernetes    auth_kubernetes_8cf36aee    n/a                        n/a
token/         token         auth_token_f9f94180         token based credentials    n/a

```

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ export SA_SECRET_NAME=$(kubectl get secrets --output=json| jq -r '.items[].metadata | select(.name|startswith("vault-auth-")).name')
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ export SA_JWT_TOKEN=$(kubectl get secret $SA_SECRET_NAME --output 'go-template={{ .data.token }}' | base64 --decode)
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ export SA_CA_CRT=$(kubectl config view --raw --minify --flatten --output 'jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ export K8S_HOST=$(kubectl config view --raw --minify --flatten --output 'jsonpath={.clusters[].cluster.server}')
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec  -it vault-0 -- vault write auth/kubernetes/config \ token_reviewer_jwt="$SA_JWT_TOKEN" \
> kubernetes_host="$K8S_HOST" \
> kubernetes_ca_cert="$SA_CA_CRT"
Success! Data written to: auth/kubernetes/config
```

Создаём политику в vault

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl cp otus-policy.hcl vault-0:./tmp/
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec -it vault-0 -- vault policy write otus-policy /tmp/otus-policy.hcl
Success! Uploaded policy: otus-policy
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus \
> bound_service_account_names=vault-auth \
> bound_service_account_namespaces=default policies=otus-policy ttl=24h
Success! Data written to: auth/kubernetes/role/otus
```

Получение токена

```/ # curl --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1713  10{     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
0   "request_id": "f04ab959-426c-ad25-f10f-a7c64725a3c3",
   "lease_id": "",
 74  "renewable": false,
9  "lease_duration": 0,
   "data": null,
 1  "wrap_info": null,
00  "warnings": null,
    "auth": {
964     "client_token": "hvs.CAESIECltA3ma9c4OcZqVOdMB0e7gl3qM3rvatWbtSYmj7reGh4KHGh2cy4zbk82NDc0NEN2NXNMc1FMUk1EaG0wWEc",
     "accessor": "wy9vDNipWXtUomJjAwVzizgu",
15    "policies": [9
36      "default",
        "otus-policy"
2    ],
05    "token_policies": [
10      "default",
       "otus-policy"
-    ],
-    "metadata": {
:      "role": "otus"-,
-      "service_account_name": "vault-auth",
:--      "service_account_namespace": "default",
 -      "service_account_secret_name": "",
-:      "service_account_uid": "8fb78ef8-b22f-477c-8430-6f74ea2604ad"
-    },
-:    "lease_duration": 86400,
--     "renewable": true,
--    "entity_id": "788128af-0cd5-d272-0bbf-3c841238493b",
:    "token_type": "service"-,
-    "orphan": true,
:--    "mfa_requirement": null,
 3    "num_uses": 06
4  }
46
}
```

Проверка токена

```
/ # curl --header "X-Vault-Token:hvs.CAESIECltA3ma9c4OcZqVOdMB0e7gl3qM3rvatWbtSYmj7reGh4KHGh2cy4zbk82NDc0NEN2NXNMc1FMUk1EaG0wWEc" $VAULT_ADDR/v1/otus/otus-ro/config
{"request_id":"e824fd3a-0aa1-65ef-d88a-efc6bd443b05","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
/ # curl --header "X-Vault-Token:hvs.CAESIECltA3ma9c4OcZqVOdMB0e7gl3qM3rvatWbtSYmj7reGh4KHGh2cy4zbk82NDc0NEN2NXNMc1FMUk1EaG0wWEc" $VAULT_ADDR/v1/otus/otus-rw/config
{"request_id":"4896eff1-a7c6-a212-139a-24572ba8b3d5","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null
```

Почему мы смогли записать otus-rw/config1 но не смогли otusrw/config?

Для записи в otus-rw нужна capability update.


Создание сертификата

```

selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec -it vault-0 -- vault write pki_int/intermediate/set-signed certificate=@/tmp/intermediate.cert.pem
WARNING! The following warnings were returned from Vault:

  * This mount hasn't configured any authority information access (AIA)
  fields; this may make it harder for systems to find missing certificates
  in the chain or to validate revocation status of certificates. Consider
  updating /config/urls or the newly generated issuer with this information.

Key                 Value
---                 -----
existing_issuers    <nil>
existing_keys       <nil>
imported_issuers    [2711c953-e379-04f5-b151-484f650ef659 fcde7a47-1cba-4cc5-0fb1-6455d7e0cc18]
imported_keys       <nil>
mapping             map[2711c953-e379-04f5-b151-484f650ef659:2b6b60da-3180-8ec9-2a89-ead2dbbec070 fcde7a47-1cba-4cc5-0fb1-6455d7e0cc18:]
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="dev.example.ru" ttl="24h"
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUTJWR7FqhHkvHLqIZ26TwGRQI0XwwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMzEyMDMxODMyNTVaFw0yODEy
MDExODMzMjVaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMLiiSkShRnm
Z6gyelfrrrioFpfoQJplLI1v3Wi0ftXQAnqwuafHX7xye3baT521dbYtlTb5hqeY
UOeor+ReT/+nCCgmFWfsCAOK+4Nqfgu4VtEkU4sKCpT2ZhfMwdFeYci2DKWK/Lao
XhUpYHwyGK95Slu6I9x8esnSq1+OiLCFfRPIQm4p5AkkCAMFuWhziIuVEVucH1IC
3InnQFmVunvVQvNlbjlQsCssmrXOPrCZ6M37NJE/YPvZBwSjjx/AHBZp8IEaLmy0
v0CGikG09TQ4ierueWvf/tTBkFUiC1Zh27v+VGhC3gfYi0H9q3p3NAoCdsk1xJEA
5V1zaFNtmW8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUPQcuKls/EevrxMU3tKquvOJqhBowHwYDVR0jBBgwFoAU
j9sBKVqvDyPPVyhx26GdHgk++bowNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
NQheM5CqUR7MLHEQwWVlQxY+ptWDKr8BN2xmthdviGrNvehTNOJ7MFLw1zbSFPcC
Tfny+nEUmaz6y4usgwuQ1hLx/R7gDY19pUG2I7FhvuIMrXdEult8r2NGaCrwwF7E
MpbPEbz+yX283mg6lxrVWVN0wDgC6tip16N08oQkiKgoPd+zc7I2COAEm6Rc1x9v
V4F+j0jmQ4MKQR8DpeZlJrUnUuvwOTPDEXzD4Er394jjRfMjiGC4a/sOWX8j7jhg
CUINGGDb5BAl+9oGKbAwuAoeaNJ6WU8BmT+ehDeKHMVZzl9bl3pf4LE7trji4P0T
Rm+/N4BPQPDv6Px90r5Tog==
-----END CERTIFICATE----- -----BEGIN CERTIFICATE-----
MIIDMjCCAhqgAwIBAgIUCGkGH0DrcIRm3JkArmtA4mtYvoEwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMzEyMDMxODE5NTBaFw0zMzEx
MzAxODIwMjBaMBUxEzARBgNVBAMTCmV4bWFwbGUucnUwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQDyNmllpz0X4P7ZHqV3sZQjYyJGxh9wNcLEm9mIxCsH
eW4GmYYuE7SQPW0/37wx/ODajyMK6GSLJkgYCQ3mX16LUS6UWQySAETrBpW/7YJC
PzqiY2v2NFpeqQbxESy97LRoM/IG21MNO8BB7HtTiHOIRHIwLpui+BzDVFD+EF5Q
YYrKo+tIeL0RWwy++V1rULSnSAbvFHVK66iXq+KaoESAcW1D3swxB07W8hbCJkvg
mPvpqLLSyo0fl7iv3TmvCzI9UFb99fsnGTNYgSfB3h3sm81gKFOcNSrn/kymqOUd
tCljaLW//PZGSDw14J9UsMfa9PwEoGE/q+BkGeEJ7427AgMBAAGjejB4MA4GA1Ud
DwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBSP2wEpWq8PI89X
KHHboZ0eCT75ujAfBgNVHSMEGDAWgBSP2wEpWq8PI89XKHHboZ0eCT75ujAVBgNV
HREEDjAMggpleG1hcGxlLnJ1MA0GCSqGSIb3DQEBCwUAA4IBAQC+nyr7nOBtTgBl
hLgq+1wsOIU50pN9SHolBdCiaCgsqaGh7/g0O1KGlpiMCi0FUXkZWRbOtkFYA/sa
RvAkT8nbVKOLmknnIlGShqWyy37XAS7eXOMUM4itvrBvlMond67pGtqSD48a363Y
70mPriM/UTKgfRXhGEXnGWkJjSUJ9mjMyaSxQfju0lROAPbApU6PZR1+ZvxhdAe0
JrFLSFoanWP8TyKKt0uV+hxt1hcFvfF66324v0doI+tvvoxJlLVwpYlzRQWTl106
GBBYI0+lIYJiQeJ39rpoC3H9ltcSUgoNezmk9rJJ98zCK3G66r7TGnvi/53qO7xg
yZ4UHi+f
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDYTCCAkmgAwIBAgIUJKNnx0fYN4q56QunoJK6SJ04ziMwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIzMTIwMzE4MzMyM1oXDTIzMTIwNDE4MzM1M1owGTEXMBUGA1UEAxMOZGV2
LmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCxFbnO
/rdF0L7c1k+5kPFR0XrkfFHTMAUkJhE6l8LG0WjFxWOkKwAe8W8j1qSY39H4KgJq
+kJLevwkjJG8CNavl0NQkog6k6QdU8jJrrzfcV1uc4tk4YP4PiXBBPETaCUixHGA
6L9jT8mx0/sDer5YcZFIjPHH8mSUhQxRAv1XxUr0Y9BBnUhP/LPN0Tw963yOm+5A
skp5PpRtGMefzGIW4PoJY9pDjiPwIBvvTA9K3GcwSvRF68SI8aUpsXcMW3jReyct
2o8M9dGRVUB6NyckwujlclDOkrTdHXpLcaX2IzrMlTZcrBixthk0bu/tP13aYhDK
d9jHeU8kS0Q2O6hzAgMBAAGjgY0wgYowDgYDVR0PAQH/BAQDAgOoMB0GA1UdJQQW
MBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUdM/KsTo3hr3wgOX7Q7FQ
yHj1MQkwHwYDVR0jBBgwFoAUPQcuKls/EevrxMU3tKquvOJqhBowGQYDVR0RBBIw
EIIOZGV2LmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAKTOEedz6M+5W5U7
qEKy63yhulMFFB1mp2kO8udYGkg5tZ7bKoG4WX7fcfXYJmrm31xza1LTYdp5GtVu
6dB58eEDvyTX+7fouf6sFXvqrDPcs3G9zNpEHUqJoCWvl5POS+lcwykEc4Yinhq7
YICueXk1kXIEpC1aoIxzevWu2Ymb5gwMk2NCi6+njKMH/vVUAoA1WeQLBX1Pq688
kO3fnC6Q6l9V16WYk3bte1O8yJfJTsCTjIELwqOPCUZmCwy28vftiqUEtkZfLGC1
mMEysTnZwWMbU9oxLwW+RnaAmlDG9CXe7/NWiRJkPK+LqvOZcHupTcan8TeBPJSp
+6p5Kb4=
-----END CERTIFICATE-----
expiration          1701714833
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUTJWR7FqhHkvHLqIZ26TwGRQI0XwwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMzEyMDMxODMyNTVaFw0yODEy
MDExODMzMjVaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMLiiSkShRnm
Z6gyelfrrrioFpfoQJplLI1v3Wi0ftXQAnqwuafHX7xye3baT521dbYtlTb5hqeY
UOeor+ReT/+nCCgmFWfsCAOK+4Nqfgu4VtEkU4sKCpT2ZhfMwdFeYci2DKWK/Lao
XhUpYHwyGK95Slu6I9x8esnSq1+OiLCFfRPIQm4p5AkkCAMFuWhziIuVEVucH1IC
3InnQFmVunvVQvNlbjlQsCssmrXOPrCZ6M37NJE/YPvZBwSjjx/AHBZp8IEaLmy0
v0CGikG09TQ4ierueWvf/tTBkFUiC1Zh27v+VGhC3gfYi0H9q3p3NAoCdsk1xJEA
5V1zaFNtmW8CAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUPQcuKls/EevrxMU3tKquvOJqhBowHwYDVR0jBBgwFoAU
j9sBKVqvDyPPVyhx26GdHgk++bowNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
NQheM5CqUR7MLHEQwWVlQxY+ptWDKr8BN2xmthdviGrNvehTNOJ7MFLw1zbSFPcC
Tfny+nEUmaz6y4usgwuQ1hLx/R7gDY19pUG2I7FhvuIMrXdEult8r2NGaCrwwF7E
MpbPEbz+yX283mg6lxrVWVN0wDgC6tip16N08oQkiKgoPd+zc7I2COAEm6Rc1x9v
V4F+j0jmQ4MKQR8DpeZlJrUnUuvwOTPDEXzD4Er394jjRfMjiGC4a/sOWX8j7jhg
CUINGGDb5BAl+9oGKbAwuAoeaNJ6WU8BmT+ehDeKHMVZzl9bl3pf4LE7trji4P0T
Rm+/N4BPQPDv6Px90r5Tog==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAsRW5zv63RdC+3NZPuZDxUdF65HxR0zAFJCYROpfCxtFoxcVj
pCsAHvFvI9akmN/R+CoCavpCS3r8JIyRvAjWr5dDUJKIOpOkHVPIya6833FdbnOL
ZOGD+D4lwQTxE2glIsRxgOi/Y0/JsdP7A3q+WHGRSIzxx/JklIUMUQL9V8VK9GPQ
QZ1IT/yzzdE8Pet8jpvuQLJKeT6UbRjHn8xiFuD6CWPaQ44j8CAb70wPStxnMEr0
RevEiPGlKbF3DFt40XsnLdqPDPXRkVVAejcnJMLo5XJQzpK03R16S3Gl9iM6zJU2
XKwYsbYZNG7v7T9d2mIQynfYx3lPJEtENjuocwIDAQABAoIBAQCfyUnKxD2dCnle
DScM+wM0338zMhYnKFpLPuom449GFOikI7MADCjkwteVD/WfV74/XbCm1MADGarw
U8KgV51X/XYo+r9fk57vM42mpjwYplM2+Z1a3r5UvccVPp9E8qEnmPgN6HXhZ7pH
8k252wRsC7WbMEpuL4KgHNl7M+ZjTcqlY6NchR7Sb/js3T5Nf0iWNKI1lGylSNgT
96dJ06fnvqTSYy28K9pHi1Lg7OgSSLQs0Nz7s+rTsqcImDXoA3xYe8gp2xiBrdc9
afBvbsBtkZjZoxA0mgUPH9iI15Sw8ReAaYjzkOWgkU5tAzwhw7bjC0JvPBUKS2kD
60NpRCAxAoGBAOHioFbOOXzULNMFV5BFyiSngduhoL3Y0ygKrY7IhrCmdfZe24wU
BABYIUJNMXhDH8yC9dLvAYVVDQMAQMFdGISkePeT9fhRR0Rhf4EihAd+qriAPJC3
t7Faec33tVm78/KHa1OSimdffsjtbaUIbK4p3GY8WhE7gJqoxgO4or8bAoGBAMix
j8PBHKvEMw0juQx5Y78pgBKaZikAuftIpiTdTYnfu8nCoMpG9jH4CJJHwKuzY9SF
HGTrFRxyOtBFElwQJh/7XLRDUaQaMDzwCVv+MxVGYxsy/RwAWhzRRet30WHkXbJe
jbIOhMnqviSKEF2jhP+IIZbAdhUzO2xkrdk8SVmJAoGAVoIfm/8Q3zC3Ff4Gwfco
ao9IWV/2Gp8Oh1hHjdZYVxD5Pmintmb3/VXDLww3NPKoG//Pu3/TWkfvWsXfBu7r
c+k1dsPQwNAH9jVMypz4aZJmOZDLITVrAV5AJdSHPJ2R2MFqJjCKFvroqHTduAWY
8b6QbQsSB2V9ZD3c0BIHKh8CgYByYG+UmqwiYFDP/jnqGAx218n70C7E03sq8L5v
aAhWuUGmvNsyLLsGw1rvMyFlOXl9ltcV1LxVV+yY4aSS/0kbFQBCY9NVeO9g61QK
L5chWtoEmEyT9sdkgQgeKE0WQzX6/9Q1U/ztrnDrFhw5oYWctBKgfdNORcJqBf7m
PWt4MQKBgQCbROT5FgmIgiiSyx0RgHnsSPeWzXGoTdmLivwopF8uw0FPcauwKKjH
2Gd7v6nY16hXh5xXseIuYomkToHl7M66d6vAReUb3hzAH4v3PAA96thU6nmRLm0m
DEDISWVYkURJedB6il0t9CPpoyq+Vtb0Z/jRo+zczdQMF10ZNmPgog==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       24:a3:67:c7:47:d8:37:8a:b9:e9:0b:a7:a0:92:ba:48:9d:38:ce:2
```

Выполнено задание со *

Реализовать доступ к vault через https

По инструкции https://akjamie.github.io/post/2023-03-25-vault-on-k8s/

```
selcov@ubuntu:~/k8s/elcovstas_platform/kubernetes-vault$ kubectl port-forward vault-0 8200:8200

selcov@ubuntu:~$ curl http://127.0.0.1:8200
Client sent an HTTP request to an HTTPS server.

selcov@ubuntu:~$ curl --insecure --header "X-Vault-Token:hvs.CAESIECltA3ma9c4OcZqVOdMB0e7gl3qM3rvatWbtSYmj7reGh4KHGh2cy4zbk82NDc0NEN2NXNMc1FMUk1EaG0wWEc"  https://127.0.0.1:8200/v1/otus/otus-ro/config
{"request_id":"9e8b5019-3cd5-8f60-6a78-75f8d1ece175","lease_id":"","renewable":false,"lease_duration":2764800,"data":{"password":"asajkjkahs","username":"otus"},"wrap_info":null,"warnings":null,"auth":null}
```
# Домашняя работа 12. Развертывание системы хранения данных

1) Добавление CRD снапшотов
2) Установка snapshot-controller
3) Установка CSI Host Path Driver
4) Создание StorageClass, storage-pvc, storage-pod

Результат:

```
root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-storage# kubectl get pod storage-pod
NAME          READY   STATUS    RESTARTS   AGE
storage-pod   1/1     Running   0          133m
root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-storage# kubectl get pvc storage-pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
storage-pvc   Bound    pvc-d1f41296-0f88-402c-8e8e-bd0ba863fc16   1Gi        RWO            csi-hostpath-sc   136m

root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-storage# kubectl exec storage-pod -it /bin/sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ #
/ #
/ # ls -l
total 44
drwxr-xr-x    2 root     root         12288 Dec  4 19:45 bin
drwxr-xr-x    2 root     root          4096 Dec 14 13:15 data
drwxr-xr-x    5 root     root           360 Dec 14 13:19 dev
drwxr-xr-x    1 root     root          4096 Dec 14 13:19 etc
drwxr-xr-x    2 nobody   nobody        4096 Dec  4 19:45 home
drwxr-xr-x    2 root     root          4096 Dec  4 19:45 lib
lrwxrwxrwx    1 root     root             3 Dec  4 19:45 lib64 -> lib
dr-xr-xr-x  361 root     root             0 Dec 14 13:19 proc
drwx------    1 root     root          4096 Dec 14 13:20 root
dr-xr-xr-x   13 root     root             0 Dec 14 13:19 sys
drwxrwxrwt    2 root     root          4096 Dec  4 19:45 tmp
drwxr-xr-x    4 root     root          4096 Dec  4 19:45 usr
drwxr-xr-x    1 root     root          4096 Dec 14 13:19 var
```

Папка data добавлена в pod

# Домашняя работа 13. ДЗ. Отладка и тестирование в Kubernetes

1) Установка kubectl-debug

Установку приложения выполнил с помощью скрипта kubernetes_debug_install.sh

```
PLUGIN_VERSION=0.1.1
# linux x86_64
curl -Lo kubectl-debug.tar.gz https://github.com/aylei/kubectl-debug/releases/download/v${PLUGIN_VERSION}/kubectl-debug_${PLUGIN_VERSION}_linux_amd64.tar.gz

tar -zxvf kubectl-debug.tar.gz kubectl-debug
sudo mv kubectl-debug /usr/local/bin/
```

Агента поставил через манифест

```
kubectl apply -f https://raw.githubusercontent.com/aylei/kubectl-debug/master/scripts/agent_daemonset.yml


root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-debug# kubectl get daemonset
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
debug-agent   3         3         3       3            3           <none>          81m
rng           3         3         3       3            3           <none>          192d

```

2) Проверка kubectl-debug

Проверил с помощью команды 

```
kubectl debug -it web --image=nicolaka/netshoot:latest --target=web

 web  ~  ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   29 101       0:00 nginx: worker process
   30 101       0:00 nginx: worker process
   31 101       0:00 nginx: worker process
   32 101       0:00 nginx: worker process
  173 root      0:02 zsh
  240 root      0:00 ps aux

 web  ~  strace -p 1 -c
strace: attach: ptrace(PTRACE_SEIZE, 1): Operation not permitted
```

К, сожалению, добавление securitycontent в pod не сработало.

Если, использовать образ alpine и установить strace, то strace работает, но процесс в контейнере не нашего nginx.

```
root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-debug# kubectl debug -it web --image=alpine target=web
Defaulting debug container name to debugger-b4lpr.
If you don't see a command prompt, try pressing enter.
/ # ps afx
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 ps afx
/ # apk add strace
(1/7) Installing zstd-libs (1.5.5-r8)
(2/7) Installing libelf (0.190-r1)
(3/7) Installing libbz2 (1.0.8-r6)
(4/7) Installing musl-fts (1.2.7-r6)
(5/7) Installing xz-libs (5.4.5-r0)
(6/7) Installing libdw (0.190-r1)
(7/7) Installing strace (6.6-r0)
Executing busybox-1.36.1-r15.trigger
OK: 10 MiB in 22 packages
/ # strace -p 1 -c
strace: Process 1 attached
^Cstrace: Process 1 detached
```


3) Установка iptables-tailes и netperf-operator

Установить netperf-operator на кластере v1.25.9 не получилось, т.к. проект уже старый не поддерживается.

```
root@knd-test-kub-m1:~/elcovstas_platform/kubernetes-debug/kit# kubectl create -f crd.yaml
error: resource mapping not found for name: "netperfs.app.example.com" namespace: "" from "crd.yaml": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
ensure CRDs are installed first
```

kube-iptables-tailer тоже устаревший репотизорий. Вижу, что есть pull в 2022 году, про поддержку версии 1.19.10 https://github.com/box/kube-iptables-tailer/pull/37

Устанавливать старую версию кластера, для этого дела не собираюсь. Прошу обновить ДЗ по этому уроку, для актуальных утилит в данное время.


# Домашняя работа 14. ДЗ. Подходы к развёртыванию кластера k8s

1) Установка и проверка кластера k8s версии 1.23

Кластер установлен по инструкции в ДЗ

Вывод команды kubectl get nodes

```
admin@master:~$ kubectl get nodes
NAME       STATUS   ROLES                  AGE     VERSION
master     Ready    control-plane,master   4m42s   v1.23.0
worker-1   Ready    <none>                 116s    v1.23.0
worker-2   Ready    <none>                 109s    v1.23.0
worker-3   Ready    <none>                 105s    v1.23.0
```

Проверим кластер с помощью установки nginx в качестве deployment

```
cat > dep.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF

kubectl apply -f dep.yaml

kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 26 Dec 2023 11:04:45 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.14.2
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-9456bbbf9 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  92s   deployment-controller  Scaled up replica set nginx-deployment-9456bbbf9 to 2
```

2) Обновление кластера до версии 1.24

Обновление компонент k8s на мастер ноде:

Сначало обновляем:
kube-apiserver
kube-controller-manager
kube-scheduler
kube-proxy
CoreDNS
etcd

```
root@master:/home/admin# sudo su -
root@master:~# apt update
Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease
Hit:4 https://download.docker.com/linux/ubuntu focal InRelease
Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:6 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Building dependency tree
Reading state information... Done
3 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@master:~# apt-cache madison kubeadm
   kubeadm |  1.28.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.28.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.28.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.27.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.26.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.25.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.25.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.25.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.25.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.25.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.25.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.23.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.23.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.22.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.22.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.21.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.21.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.21.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.21.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.21.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.21.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.20.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.20.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.19.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.19.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.20-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.19-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.18-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.18.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.4-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.18.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.17.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.7-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.17.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.11-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.16.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.16.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.15.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.15.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.15.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.15.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.14.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.14.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.13.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.13.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.13.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.13.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.12.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.12.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.11.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.11.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.10.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.10.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.10.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.10.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.10.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.9.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.9.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.9.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.8.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.1-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.0-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.8.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.7.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.7.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.7.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.7.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.7.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.3-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.7.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.6.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.6.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.6.11-01 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.6.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.6.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |   1.5.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
root@master:~# apt-cache madison kubeadm | grep 1.24
   kubeadm | 1.24.17-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.16-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.15-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.14-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.13-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.12-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.11-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm | 1.24.10-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.9-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.8-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.7-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.6-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.5-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.4-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.3-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.2-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.1-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
   kubeadm |  1.24.0-00 | https://apt.kubernetes.io kubernetes-xenial/main amd64 Packages
root@master:~# apt-mark unhold kubeadm && \
> apt-get update && apt-get install -y kubeadm=1.24.17-00 && \
> apt-mark hold kubeadm
Canceled hold on kubeadm.
Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:5 https://download.docker.com/linux/ubuntu focal InRelease
Hit:6 https://packages.cloud.google.com/apt kubernetes-xenial InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
Need to get 9,196 kB of archives.
After this operation, 348 kB of additional disk space will be used.
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.24.17-00 [9,196 kB]
Fetched 9,196 kB in 1s (8,503 kB/s)
(Reading database ... 105657 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.24.17-00_amd64.deb ...
Unpacking kubeadm (1.24.17-00) over (1.23.0-00) ...
Setting up kubeadm (1.24.17-00) ...
kubeadm set on hold.
root@master:~# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W1226 11:41:08.717404   21127 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/containerd/containerd.sock". Please update your configuration!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.23.0
[upgrade/versions] kubeadm version: v1.24.17
I1226 11:41:12.319807   21127 version.go:256] remote version is much newer: v1.29.0; falling back to: stable-1.24
[upgrade/versions] Target version: v1.24.17
[upgrade/versions] Latest version in the v1.23 series: v1.23.17

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     4 x v1.23.0   v1.23.17

Upgrade to the latest version in the v1.23 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.23.0   v1.23.17
kube-controller-manager   v1.23.0   v1.23.17
kube-scheduler            v1.23.0   v1.23.17
kube-proxy                v1.23.0   v1.23.17
CoreDNS                   v1.8.6    v1.8.6
etcd                      3.5.1-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.23.17

_____________________________________________________________________

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     4 x v1.23.0   v1.24.17

Upgrade to the latest stable version:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.23.0   v1.24.17
kube-controller-manager   v1.23.0   v1.24.17
kube-scheduler            v1.23.0   v1.24.17
kube-proxy                v1.23.0   v1.24.17
CoreDNS                   v1.8.6    v1.8.6
etcd                      3.5.1-0   3.5.6-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.24.17

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

root@master:~# kubeadm upgrade apply v1.24.17
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W1226 11:41:17.069748   21221 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/containerd/containerd.sock". Please update your configuration!
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.24.17"
[upgrade/versions] Cluster version: v1.23.0
[upgrade/versions] kubeadm version: v1.24.17
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.24.17" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-12-26-11-42-39/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=etcd
[upgrade/staticpods] Component "etcd" upgraded successfully!
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests3269519028"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-12-26-11-42-39/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-12-26-11-42-39/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-12-26-11-42-39/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upgrade/postupgrade] Removing the deprecated label node-role.kubernetes.io/master='' from all control plane Nodes. After this step only the label node-role.kubernetes.io/control-plane='' will be present on control plane Nodes.
[upgrade/postupgrade] Adding the new taint &Taint{Key:node-role.kubernetes.io/control-plane,Value:,Effect:NoSchedule,TimeAdded:<nil>,} to all control plane Nodes. After this step both taints &Taint{Key:node-role.kubernetes.io/control-plane,Value:,Effect:NoSchedule,TimeAdded:<nil>,} and &Taint{Key:node-role.kubernetes.io/master,Value:,Effect:NoSchedule,TimeAdded:<nil>,} should be present on control plane Nodes.
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.24.17". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

Проверяем версии 

```
root@master:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.17", GitCommit:"22a9682c8fe855c321be75c5faacde343f909b04", GitTreeState:"clean", BuildDate:"2023-08-23T23:43:11Z", GoVersion:"go1.20.7", Compiler:"gc", Platform:"linux/amd64"}
root@master:~# kubelet --version
Kubernetes v1.23.0
root@master:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
```

Теперь нужно обновить kubelet  и kubectl

Обновление kubelet

```
sudo su -
apt update
apt-cache madison kubelet
apt-cache madison kubelet | grep 1.24
apt-mark unhold kubelet && \
apt-get update && apt-get install -y kubelet=1.24.17-00 && \
apt-mark hold kubelet
kubeadm upgrade plan
kubeadm upgrade apply v1.24.17
systemctl daemon-reload
systemctl restart kubelet.service
```

Обновление kubectl

```
sudo su -
apt update
apt-cache madison kubectl
apt-cache madison kubectl | grep 1.24
apt-mark unhold kubectl && \
apt-get update && apt-get install -y kubectl=1.24.17-00 && \
apt-mark hold kubectl
kubeadm upgrade plan
kubeadm upgrade apply v1.24.17
```

Проверка версии

```
admin@master:~$ kubelet --version
Kubernetes v1.24.17
admin@master:~$ kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.17", GitCommit:"22a9682c8fe855c321be75c5faacde343f909b04", GitTreeState:"clean", BuildDate:"2023-08-23T23:44:35Z", GoVersion:"go1.20.7", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.4
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.17", GitCommit:"22a9682c8fe855c321be75c5faacde343f909b04", GitTreeState:"clean", BuildDate:"2023-08-23T23:37:25Z", GoVersion:"go1.20.7", Compiler:"gc", Platform:"linux/amd64

admin@master:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
master     Ready    control-plane   67m   v1.24.17
worker-1   Ready    <none>          64m   v1.23.0
worker-2   Ready    <none>          64m   v1.23.0
worker-3   Ready    <none>          64m   v1.23.0

```

Теперь обновляем worker node кластера

Выводим ноду из кластера

```
admin@master:~$ kubectl drain worker-1 --ignore-daemonsets
node/worker-1 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-4k2jb, kube-system/kube-proxy-2mmw2
evicting pod kube-system/coredns-57575c5f89-7x8td
evicting pod default/nginx-deployment-9456bbbf9-2wxhn
pod/nginx-deployment-9456bbbf9-2wxhn evicted
pod/coredns-57575c5f89-7x8td evicted
node/worker-1 drained
```

Проверяем статус

```
admin@master:~$ kubectl get nodes
NAME       STATUS                     ROLES           AGE   VERSION
master     Ready                      control-plane   72m   v1.24.17
worker-1   Ready,SchedulingDisabled   <none>          69m   v1.23.0
worker-2   Ready                      <none>          69m   v1.23.0
worker-3   Ready                      <none>          69m   v1.23.0

```

Обновляем worker ноду

```
sudo su -
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.24.17-00 && \
apt-mark hold kubeadm
sudo kubeadm upgrade node
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.24.17-00 kubectl=1.24.17-00 && \
apt-mark hold kubelet kubectl
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```


Возвращение ноды в  планирование  и проверяем версии:

```
admin@master:~$ kubectl uncordon worker-1
node/worker-1 uncordoned
admin@master:~$ kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
master     Ready    control-plane   77m   v1.24.17
worker-1   Ready    <none>          74m   v1.24.17
worker-2   Ready    <none>          74m   v1.23.0
worker-3   Ready    <none>          74m   v1.23.0

```

Обновляем по аналогии другие worker ноды. 
Вывод kubectl get nodes

```
admin@master:~$ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION    INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
master     Ready    control-plane   83m   v1.24.17   10.129.0.32   <none>        Ubuntu 20.04.6 LTS   5.4.0-169-generic   containerd://1.6.26
worker-1   Ready    <none>          80m   v1.24.17   10.129.0.7    <none>        Ubuntu 20.04.6 LTS   5.4.0-169-generic   containerd://1.6.26
worker-2   Ready    <none>          80m   v1.24.17   10.129.0.16   <none>        Ubuntu 20.04.6 LTS   5.4.0-169-generic   containerd://1.6.26
worker-3   Ready    <none>          80m   v1.24.17   10.129.0.19   <none>        Ubuntu 20.04.6 LTS   5.4.0-169-generic   containerd://1.6.26
```

3) Установка кластера с помощью kubespray

Скачал репотизорий kubespray, но при выполнении команды sudo pip install -r requirements.txt  столкнулся с ошибкой совместимости версий.

```
ERROR: Ignored the following yanked versions: 9.0.0
ERROR: Ignored the following versions that require a different python version: 7.0.0 Requires-Python >=3.9; 7.0.0a1 Requires-Python >=3.9.0; 7.0.0a2 Requires-Python >=3.9; 7.0.0b1 Requires-Python >=3.9; 7.0.0rc1 Requires-Python >=3.9; 7.1.0 Requires-Python >=3.9; 7.2.0 Requires-Python >=3.9; 7.3.0 Requires-Python >=3.9; 7.4.0 Requires-Python >=3.9; 7.5.0 Requires-Python >=3.9; 7.6.0 Requires-Python >=3.9; 7.7.0 Requires-Python >=3.9; 8.0.0 Requires-Python >=3.9; 8.0.0a1 Requires-Python >=3.9; 8.0.0a2 Requires-Python >=3.9; 8.0.0a3 Requires-Python >=3.9; 8.0.0b1 Requires-Python >=3.9; 8.0.0rc1 Requires-Python >=3.9; 8.1.0 Requires-Python >=3.9; 8.2.0 Requires-Python >=3.9; 8.3.0 Requires-Python >=3.9; 8.4.0 Requires-Python >=3.9; 8.5.0 Requires-Python >=3.9; 8.6.0 Requires-Python >=3.9; 8.6.1 Requires-Python >=3.9; 8.7.0 Requires-Python >=3.9; 9.0.1 Requires-Python >=3.10; 9.1.0 Requires-Python >=3.10
ERROR: Could not find a version that satisfies the requirement ansible==8.5.0 (from versions: 1.0, 1.1, 1.2, 1.2.1, 1.2.2, 1.2.3, 1.3.0, 1.3.1, 1.3.2, 1.3.3, 1.3.4, 1.4, 1.4.1, 1.4.2, 1.4.3, 1.4.4, 1.4.5, 1.5, 1.5.1, 1.5.2, 1.5.3, 1.5.4, 1.5.5, 1.6, 1.6.1, 1.6.2, 1.6.3, 1.6.4, 1.6.5, 1.6.6, 1.6.7, 1.6.8, 1.6.9, 1.6.10, 1.7, 1.7.1, 1.7.2, 1.8, 1.8.1, 1.8.2, 1.8.3, 1.8.4, 1.9.0.1, 1.9.1, 1.9.2, 1.9.3, 1.9.4, 1.9.5, 1.9.6, 2.0.0.0, 2.0.0.1, 2.0.0.2, 2.0.1.0, 2.0.2.0, 2.1.0.0, 2.1.1.0, 2.1.2.0, 2.1.3.0, 2.1.4.0, 2.1.5.0, 2.1.6.0, 2.2.0.0, 2.2.1.0, 2.2.2.0, 2.2.3.0, 2.3.0.0, 2.3.1.0, 2.3.2.0, 2.3.3.0, 2.4.0.0, 2.4.1.0, 2.4.2.0, 2.4.3.0, 2.4.4.0, 2.4.5.0, 2.4.6.0, 2.5.0a1, 2.5.0b1, 2.5.0b2, 2.5.0rc1, 2.5.0rc2, 2.5.0rc3, 2.5.0, 2.5.1, 2.5.2, 2.5.3, 2.5.4, 2.5.5, 2.5.6, 2.5.7, 2.5.8, 2.5.9, 2.5.10, 2.5.11, 2.5.12, 2.5.13, 2.5.14, 2.5.15, 2.6.0a1, 2.6.0a2, 2.6.0rc1, 2.6.0rc2, 2.6.0rc3, 2.6.0rc4, 2.6.0rc5, 2.6.0, 2.6.1, 2.6.2, 2.6.3, 2.6.4, 2.6.5, 2.6.6, 2.6.7, 2.6.8, 2.6.9, 2.6.10, 2.6.11, 2.6.12, 2.6.13, 2.6.14, 2.6.15, 2.6.16, 2.6.17, 2.6.18, 2.6.19, 2.6.20, 2.7.0.dev0, 2.7.0a1, 2.7.0b1, 2.7.0rc1, 2.7.0rc2, 2.7.0rc3, 2.7.0rc4, 2.7.0, 2.7.1, 2.7.2, 2.7.3, 2.7.4, 2.7.5, 2.7.6, 2.7.7, 2.7.8, 2.7.9, 2.7.10, 2.7.11, 2.7.12, 2.7.13, 2.7.14, 2.7.15, 2.7.16, 2.7.17, 2.7.18, 2.8.0a1, 2.8.0b1, 2.8.0rc1, 2.8.0rc2, 2.8.0rc3, 2.8.0, 2.8.1, 2.8.2, 2.8.3, 2.8.4, 2.8.5, 2.8.6, 2.8.7, 2.8.8, 2.8.9, 2.8.10, 2.8.11, 2.8.12, 2.8.13, 2.8.14, 2.8.15, 2.8.16rc1, 2.8.16, 2.8.17rc1, 2.8.17, 2.8.18rc1, 2.8.18, 2.8.19rc1, 2.8.19, 2.8.20rc1, 2.8.20, 2.9.0b1, 2.9.0rc1, 2.9.0rc2, 2.9.0rc3, 2.9.0rc4, 2.9.0rc5, 2.9.0, 2.9.1, 2.9.2, 2.9.3, 2.9.4, 2.9.5, 2.9.6, 2.9.7, 2.9.8, 2.9.9, 2.9.10, 2.9.11, 2.9.12, 2.9.13, 2.9.14rc1, 2.9.14, 2.9.15rc1, 2.9.15, 2.9.16rc1, 2.9.16, 2.9.17rc1, 2.9.17, 2.9.18rc1, 2.9.18, 2.9.19rc1, 2.9.19, 2.9.20rc1, 2.9.20, 2.9.21rc1, 2.9.21, 2.9.22rc1, 2.9.22, 2.9.23rc1, 2.9.23, 2.9.24rc1, 2.9.24, 2.9.25rc1, 2.9.25, 2.9.26rc1, 2.9.26, 2.9.27rc1, 2.9.27, 2.10.0a1, 2.10.0a2, 2.10.0a3, 2.10.0a4, 2.10.0a5, 2.10.0a6, 2.10.0a7, 2.10.0a8, 2.10.0a9, 2.10.0b1, 2.10.0b2, 2.10.0rc1, 2.10.0, 2.10.1, 2.10.2, 2.10.3, 2.10.4, 2.10.5, 2.10.6, 2.10.7, 3.0.0b1, 3.0.0rc1, 3.0.0, 3.1.0, 3.2.0, 3.3.0, 3.4.0, 4.0.0a1, 4.0.0a2, 4.0.0a3, 4.0.0a4, 4.0.0b1, 4.0.0b2, 4.0.0rc1, 4.0.0, 4.1.0, 4.2.0, 4.3.0, 4.4.0, 4.5.0, 4.6.0, 4.7.0, 4.8.0, 4.9.0, 4.10.0, 5.0.0a1, 5.0.0a2, 5.0.0a3, 5.0.0b1, 5.0.0b2, 5.0.0rc1, 5.0.1, 5.1.0, 5.2.0, 5.3.0, 5.4.0, 5.5.0, 5.6.0, 5.7.0, 5.7.1, 5.8.0, 5.9.0, 5.10.0, 6.0.0a1, 6.0.0a2, 6.0.0a3, 6.0.0b1, 6.0.0b2, 6.0.0rc1, 6.0.0, 6.1.0, 6.2.0, 6.3.0, 6.4.0, 6.5.0, 6.6.0, 6.7.0, 9.0.0a1, 9.0.0a2, 9.0.0a3, 9.0.0b1, 9.0.0rc1)
ERROR: No matching distribution found for ansible==8.5.0
```

Будем использовать docker контейнер для этого дела

```
git checkout v2.23.1
docker pull quay.io/kubespray/kubespray:v2.23.1
docker run --rm -it --mount type=bind,source="$(pwd)"/inventory/sample,dst=/inventory \
  --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.23.1 bash
# Inside the container you may now run the kubespray playbooks:
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```

Пишем конфиг inventory

```
root@bacbb2bef9bd:/kubespray# cat inventory/sample/inventory.ini
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
node1 ansible_host=158.160.23.111 etcd_member_name=etcd1
node2 ansible_host=158.160.13.18
node3 ansible_host=51.250.97.192
node4 ansible_host=62.84.120.191
# node5 ansible_host=95.54.0.16  # ip=10.3.0.5 etcd_member_name=etcd5
# node6 ansible_host=95.54.0.17  # ip=10.3.0.6 etcd_member_name=etcd6

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
node1
# node2
# node3

[etcd]
node1
# node2
# node3

[kube_node]
node2
node3
node4
# node5
# node6

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

Запускаем роль

```
ansible-playbook -i inventory/sample/inventory.ini --become --become-user=root --user=admin --private-key /root/.ssh/id_rsa cluster.yml
```


Завершение работы роли

```
PLAY RECAP **************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node1                      : ok=751  changed=153  unreachable=0    failed=0    skipped=1283 rescued=0    ignored=8
node2                      : ok=511  changed=95   unreachable=0    failed=0    skipped=783  rescued=0    ignored=1
node3                      : ok=511  changed=95   unreachable=0    failed=0    skipped=782  rescued=0    ignored=1
node4                      : ok=511  changed=95   unreachable=0    failed=0    skipped=782  rescued=0    ignored=1

Tuesday 26 December 2023  15:54:27 +0000 (0:00:00.234)       0:15:43.928 ******
===============================================================================
download : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------------------ 72.47s
download : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------------------ 44.22s
kubernetes/kubeadm : Join to cluster ---------------------------------------------------------------------------------------------------------------------------------------------------------------- 25.30s
kubernetes/preinstall : Install packages requirements ----------------------------------------------------------------------------------------------------------------------------------------------- 24.10s
kubernetes/preinstall : Preinstall | wait for the apiserver to be running --------------------------------------------------------------------------------------------------------------------------- 23.98s
download : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------------------ 20.63s
download : Download_container | Download image if required ------------------------------------------------------------------------------------------------------------------------------------------ 13.47s
container-engine/crictl : Download_file | Download item --------------------------------------------------------------------------------------------------------------------------------------------- 12.42s
container-engine/containerd : Download_file | Download item ----------------------------------------------------------------------------------------------------------------------------------------- 12.13s
container-engine/nerdctl : Download_file | Download item -------------------------------------------------------------------------------------------------------------------------------------------- 12.07s
container-engine/runc : Download_file | Download item ----------------------------------------------------------------------------------------------------------------------------------------------- 11.65s
kubernetes/control-plane : Kubeadm | Initialize first master ---------------------------------------------------------------------------------------------------------------------------------------- 11.04s
download : Download_container | Download image if required ------------------------------------------------------------------------------------------------------------------------------------------ 10.85s
container-engine/crictl : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------------------------------- 9.02s
etcdctl_etcdutl : Download_file | Download item ------------------------------------------------------------------------------------------------------------------------------------------------------ 8.90s
container-engine/nerdctl : Download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------------------ 8.63s
container-engine/crictl : Download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------------------- 8.61s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ------------------------------------------------------------------------------------------------------------------------------------------ 8.47s
container-engine/containerd : Download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------- 8.38s
container-engine/nerdctl : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------------------------------ 8.36s
```

Вывод команды kubectl get nodes

```
root@node1:~# kubectl get nodes
NAME    STATUS   ROLES           AGE     VERSION
node1   Ready    control-plane   10m     v1.27.7
node2   Ready    <none>          9m39s   v1.27.7
node3   Ready    <none>          9m38s   v1.27.7
node4   Ready    <none>          9m37s   v1.27.7
```

4) * Выполните установку кластера с 3 master-нодами и 2 worker-нодами, можно использовать kubeadm или любой другой способ установки kubernetes.

Устанавливал с помощью rke v1.


Вывод команды kubectl get nodes

```
root@knd-test-kub-m1:~# kubectl get nodes
NAME                      STATUS   ROLES               AGE    VERSION
knd-test-kub-m1.fors.ru   Ready    controlplane,etcd   204d   v1.25.9
knd-test-kub-m2.fors.ru   Ready    controlplane,etcd   204d   v1.25.9
knd-test-kub-m3.fors.ru   Ready    controlplane,etcd   204d   v1.25.9
knd-test-kub-s1.fors.ru   Ready    worker              204d   v1.25.9
knd-test-kub-s2.fors.ru   Ready    worker              204d   v1.25.9
knd-test-kub-s3.fors.ru   Ready    worker              204d   v1.25.9
```

Инструкция по ссылке https://itisgood.ru/2020/01/29/ustanovka-proizvodstvennogo-klastera-kubernetes-s-rancher-rke/

