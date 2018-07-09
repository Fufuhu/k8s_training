
# helmの初期化

```
$ helm init
Creating /Users/01013548/.helm
Creating /Users/01013548/.helm/repository
Creating /Users/01013548/.helm/repository/cache
Creating /Users/01013548/.helm/repository/local
Creating /Users/01013548/.helm/plugins
Creating /Users/01013548/.helm/starters
Creating /Users/01013548/.helm/cache/archive
Creating /Users/01013548/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /Users/01013548/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```

これだと、~/.helm配下に全部入ってしまうので環境変数をHELM_HOMEに指定の上
実行する。

```
$ export HELM_HOME=`pwd`
$ helm init
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/repository
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/repository/cache
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/repository/local
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/plugins
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/starters
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/cache/archive
Creating /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /Users/01013548/repositories/personal/k8s_training/work/02_initialize_helm.
Warning: Tiller is already installed in the cluster.
(Use --client-only to suppress this message, or --upgrade to upgrade Tiller to the current version.)
Happy Helming!
```

ここでtillerもインストールされるので、

tillerが導入されていることを確認。

```
$ kubectl get deployments --namespace kube-system
NAME            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-dns        1         1         1            1           173d
tiller-deploy   1         1         1            1           8m
```

# チャート例の導入

```
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

`helm repo update`コマンドでここのチャートリポジトリから最新情報を収集する。

```
$ helm install stable/mysql
NAME:   belligerent-sparrow
LAST DEPLOYED: Thu Jul  5 14:39:13 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                       TYPE    DATA  AGE
belligerent-sparrow-mysql  Opaque  2     1s

==> v1/ConfigMap
NAME                            DATA  AGE
belligerent-sparrow-mysql-test  1     1s

==> v1/PersistentVolumeClaim
NAME                       STATUS  VOLUME                                    CAPACITY  ACCESS MODES  STORAGECLASS  AGE
belligerent-sparrow-mysql  Bound   pvc-bfe7376f-8015-11e8-80cb-025000000001  8Gi       RWO           hostpath      1s

==> v1/Service
NAME                       TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
belligerent-sparrow-mysql  ClusterIP  10.99.227.123  <none>       3306/TCP  1s

==> v1beta1/Deployment
NAME                       DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
belligerent-sparrow-mysql  1        1        1           0          1s

==> v1/Pod(related)
NAME                                        READY  STATUS    RESTARTS  AGE
belligerent-sparrow-mysql-8578c46d65-pp4x6  0/1    Init:0/1  0         1s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
belligerent-sparrow-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default belligerent-sparrow-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h belligerent-sparrow-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following commands to route the connection:
    export POD_NAME=$(kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 3306:3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

まずは、どんなものがデプロイされているかを確認。

```
$ kubectl get all
NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/belligerent-sparrow-mysql   1         1         1            1           5m

NAME                                      DESIRED   CURRENT   READY     AGE
rs/belligerent-sparrow-mysql-8578c46d65   1         1         1         5m

NAME                               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/belligerent-sparrow-mysql   1         1         1            1           5m

NAME                                      DESIRED   CURRENT   READY     AGE
rs/belligerent-sparrow-mysql-8578c46d65   1         1         1         5m

NAME                                            READY     STATUS    RESTARTS   AGE
po/belligerent-sparrow-mysql-8578c46d65-pp4x6   1/1       Running   0          5m

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
svc/belligerent-sparrow-mysql   ClusterIP   10.99.227.123   <none>        3306/TCP   5m
svc/kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP    173d
```
`deploy/belligerent-sparrow-mysql`からMySQL?がインストールされていることがわかる。


表示された指示にしたがってみる。

```
$ MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default belligerent-sparrow-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

$ echo $MYSQL_ROOT_PASSWORD
wX4BFJIveP
```

ちょっと掘ってみる。

```
kubectl get secret --namespace default belligerent-sparrow-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
```

この部分は、`default`名前空間に含まれている `belligerent-sparrow-mysql`のsecretリソースの情報を参照して、
jsonのパスからMySQLのルートパスワードを抜き出している。
secretリソースをgetしてもbase64でエンコードされてるので、base64コマンドでデコードする必要がある。

```
$ kubectl get secret --namespace default belligerent-sparrow-mysql
NAME                        TYPE      DATA      AGE
belligerent-sparrow-mysql   Opaque    2         17m
```

`-o json`を指定してJSON形式で`belligerent-sparrow-mysql` secretリソースの中身を表示させて見る。

```
$ kubectl get secret --namespace default belligerent-sparrow-mysql -o json
{
    "apiVersion": "v1",
    "data": {
        "mysql-password": "ajRLdHNsWE5IcQ==",
        "mysql-root-password": "d1g0QkZKSXZlUA=="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2018-07-05T05:39:13Z",
        "labels": {
            "app": "belligerent-sparrow-mysql",
            "chart": "mysql-0.8.2",
            "heritage": "Tiller",
            "release": "belligerent-sparrow"
        },
        "name": "belligerent-sparrow-mysql",
        "namespace": "default",
        "resourceVersion": "4170702",
        "selfLink": "/api/v1/namespaces/default/secrets/belligerent-sparrow-mysql",
        "uid": "bfe37419-8015-11e8-80cb-025000000001"
    },
    "type": "Opaque"
}
```

確かに`mysql-root-password`が見えます。

```
$ kubectl get secret --namespace default belligerent-sparrow-mysql -o jsonpath="{.data.mysql-root-password}"
d1g0QkZKSXZlUA==
```

jsonpathから`.data.mysql-root-password` を、引っ張れる。
ここで、`base64`コマンドでデコードする。

```
$ kubectl get secret --namespace default belligerent-sparrow-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
wX4BFJIveP
```


あとはDBへのアクセス手順。
パスワードを要求されたら上のパスワードを入れる。

```
kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il
$ apt-get update && apt-get install mysql-client -y
$ mysql -h belligerent-sparrow-mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 730
Server version: 5.7.14 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.02 sec)
```

Podを立ち上げてクラスタ内からMySQLへのアクセスはできる。


Kubernetesのクラスタ外からアクセスする場合の手順についても示してくれている。

```
To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following commands to route the connection:
    export POD_NAME=$(kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 3306:3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

一応やって行く

```
$ MYSQL_HOST=127.0.0.1
$ MYSQL_PORT=3306
```
これはMySQLのホストとポートを指定しているだけ、
ローカルのk8sを使ってるのでこれで良い。

```
$ export POD_NAME=$(kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql" -o jsonpath="{.items[0].metadata.name}")
```
PODの名前を取得している。


ちょっと中身を解説。
```
$ kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql"
NAME                                         READY     STATUS    RESTARTS   AGE
belligerent-sparrow-mysql-8578c46d65-pp4x6   1/1       Running   0          1h
```
Podの情報を取っている。


`-o json`をつけることで取得情報をJSON形式で表示

```
$ kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql" -o json
~~~~長いので省略~~~~~
```

`-o jsonpath=`でどの項目を取得するかを指定。
今回はpodの名前を取得

```
$ kubectl get pods --namespace default -l "app=belligerent-sparrow-mysql" -o jsonpath="{.items[0].metadata.name}"
belligerent-sparrow-mysql-8578c46d65-pp4x6
```


```
kubectl port-forward $POD_NAME 3306:3306
```

ポートフォワーディングして外からのアクセスに対応。

ここまでするとローカルからmysqlクライアントを使って接続で切る。

```
$ mysql -h 127.0.0.1 -P3306 -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 915
Server version: 5.7.14 MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.02 sec)
```

# helm観点で色々見て見る

```
$ helm ls
NAME                    REVISION        UPDATED                         STATUS          CHART           NAMESPACE
belligerent-sparrow     1               Thu Jul  5 14:39:13 2018        DEPLOYED        mysql-0.8.2     default
```

`belligerent-sparrow`が入っている。
これのステータスを見る。

```
$ helm status belligerent-sparrow
LAST DEPLOYED: Thu Jul  5 14:39:13 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                       TYPE    DATA  AGE

〜〜〜〜〜〜〜中略〜〜〜〜〜〜〜〜

==> v1/Pod(related)
NAME                                        READY  STATUS   RESTARTS  AGE
belligerent-sparrow-mysql-8578c46d65-pp4x6  1/1    Running  0         1h


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:

〜〜〜〜〜〜〜〜中略〜〜〜〜〜〜〜〜

    kubectl port-forward $POD_NAME 3306:3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

`belligerent-sparrow`を削除する。

```
$ helm delete belligerent-sparrow
release "belligerent-sparrow" deleted
```

statusを確認すると、
`STATUS: DELETED`から削除済みだることがわかる。

```
$ helm status belligerent-sparrow
LAST DEPLOYED: Thu Jul  5 14:39:13 2018
NAMESPACE: default
STATUS: DELETED

NOTES:
〜〜〜〜略〜〜〜〜

$ kubectl get all

NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
svc/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   173d
```

PODなども綺麗さっぱり消えているが、Audit目的で履歴は残っているので、
`helm rollback`で削除を取り消すこともできる。