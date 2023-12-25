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
