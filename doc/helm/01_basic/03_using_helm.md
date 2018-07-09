
# 概要

- Helmチャートを探す
  - `helm search`
- 特定チャートの詳細を確認する
  - `helm search チャート名(一部のみ可)`
  - `helm inspect チャート名`
- Helmパッケージをインストールする
  - `helm install チャート名`
- 特定リリースの状態を確認する
  - `helm status リリース名`
- チャートをカスタマイズする
  - `helm inspect values チャート名`
  - `helm install -f 設定ファイル名.yaml チャート名`
- 色々なインストール手段
  - `helm install 〜〜〜`
- リリースのアップグレード・ロールバック
  - `helm upgrade リリース名 チャート名`
  - `helm rollback リリース名 リビジョン`
- リリースの削除
  - `helm delete リリース名`
- リリースの確認
  - `helm list`
  - `helm list --deleted`
  - `helm list -all`

# Helmチャートを探す

```
$ helm search
NAME                                    CHART VERSION   APP VERSION                     DESCRIPTION
stable/acs-engine-autoscaler            2.2.0           2.1.1                           Scales worker nodes within agent pools
stable/aerospike                        0.1.7           v3.14.1.2                       A Helm chart for Aerospike in Kubernetes
stable/anchore-engine                   0.1.7           0.1.10                          Anchore container analysis and policy evaluatio...
stable/apm-server                       0.1.0           6.2.4                           The server receives data from the Elastic APM a...
stable/ark                              1.0.1           0.8.2                           A Helm chart for ark
stable/artifactory                      7.2.1           6.0.0                           Universal Repository Manager supporting all maj...
stable/artifactory-ha                   0.2.1           6.0.0                           Universal Repository Manager supporting all maj...
stable/auditbeat                        0.1.0           6.2.4                           A lightweight shipper to audit the activities o...
```

# 特定チャートの詳細を確認する

```
$ helm search mysql
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/mysql                            0.8.2           5.7.14          Fast, reliable, scalable, and easy to use open-...
stable/prometheus-mysql-exporter        0.1.0           v0.10.0         A Helm chart for prometheus mysql exporter with...
stable/percona                          0.3.2           5.7.17          free, fully compatible, enhanced, open source d...
stable/percona-xtradb-cluster           0.1.5           5.7.19          free, fully compatible, enhanced, open source d...
stable/phpmyadmin                       0.1.6           4.8.0           phpMyAdmin is an mysql administration frontend
stable/gcloud-sqlproxy                  0.3.6           1.11            Google Cloud SQL Proxy
stable/mariadb                          4.2.5           10.1.34         Fast, reliable, scalable, and easy to use open-...
```

```
$ helm inspect stable/mysql
appVersion: 5.7.14
description: Fast, reliable, scalable, and easy to use open-source relational database
  system.
engine: gotpl
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
- mysql
- database
- sql
maintainers:
- email: viglesias@google.com
  name: viglesiasce
name: mysql
sources:
〜〜〜〜めちゃ長いので割愛〜〜〜〜
```


# Helmパッケージをインストールする

```
helm install stable/mariadb
Fetched stable/mariadb-0.3.0 to /Users/mattbutcher/Code/Go/src/k8s.io/helm/mariadb-0.3.0.tgz
happy-panda
Last Deployed: Wed Sep 28 12:32:28 2016
Namespace: default
Status: DEPLOYED
〜〜〜〜割愛〜〜〜〜
```

# 特定リリースの状態を確認する

```
$ helm status happy-panda
Last Deployed: Wed Sep 28 12:32:28 2016
Namespace: default
Status: DEPLOYED

Resources:
==> v1/Service
NAME                     CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
happy-panda-mariadb   10.0.0.70    <none>        3306/TCP   4m
〜〜〜〜割愛〜〜〜〜
```

# チャートをカスタマイズする

```
$ helm inspect values stable/mariadb
Fetched stable/mariadb-0.3.0.tgz to /Users/mattbutcher/Code/Go/src/k8s.io/helm/mariadb-0.3.0.tgz
## Bitnami MariaDB image version
## ref: https://hub.docker.com/r/bitnami/mariadb/tags/
##
## Default: none
imageTag: 10.1.14-r3

## Specify a imagePullPolicy
## Default to 'Always' if imageTag is 'latest', else set to 'IfNotPresent'
## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
##
# imagePullPolicy:

## Specify password for root user
## ref: https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#setting-the-root-password-on-first-run
##
# mariadbRootPassword:
〜〜〜〜割愛〜〜〜〜
```

```
$ echo '{mariadbUser: user0, mariadbDatabase: user0db}' > config.yaml
$ helm install -f config.yaml stable/mariadb
```

# 色々なインストール手段

|コマンド|内容|
|---|---|
|`helm install チャート名`| チャートリポジトリからのインストール|
|`helm install ファイル名.tgz` | ローカルのアーカイブからのインストール|
|`helm install path/to/foo` | パッケージされてないチャートディレクトリからのインストール |
|`helm install https://URL` | 特定のURLに置かれたアーカイブからのインストール |