# Helmの紹介と活用ポイント

## 自己紹介

## helmとは

![](2018-08-18-10-25-54.png)

- helmとは
    - kubernetesのパッケージマネージャ
    - Debian系ディストリビューションのapt, RHEL系のyumのk8s環境版みたいなもの
    - [公式ページ](https://helm.sh/)
    - [リポジトリ](https://github.com/helm/helm)

## helmの基本機能

- リポジトリからのチャート取得
- チャート間の依存関係の解決
- 独自チャート作成記法の提供

※ チャート = Debian/RHELにおけるパッケージ

## 軽くお試し

### maridadbのインストール

```bash
$ helm install stable/mariadb --name demo -f env.yaml
```

#### 補足(env.yaml)

env.yaml

```yaml
service:
  type: NodePort
  port: 30001

rootUser:
    password: "testtesttest"
    forcePassword: "testtesttest"

master:
  persistence:
    enabled: false

replication:
  enabled: false
```

###　動作確認

```bash
$ kubectl get all
NAME                 READY     STATUS    RESTARTS   AGE
pod/demo-mariadb-0   0/1       Running   0          19s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
service/demo-mariadb   NodePort    10.43.103.122   <none>        30001:31809/TCP   19s
service/kubernetes     ClusterIP   10.43.0.1       <none>        443/TCP           35d
```

```bash
$ mysql -h 127.0.0.1 -P 31809 -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.5.5-10.1.35-MariaDB Source distribution

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit;
```

## helmで独自チャートを作る

### helmチャートのファイル構成

```
ベースディレクトリ/
  Chart.yaml          # A YAML file containing information about the chart
  LICENSE             # OPTIONAL: A plain text file containing the license for the chart
  README.md           # OPTIONAL: A human-readable README file
  requirements.yaml   # OPTIONAL: A YAML file listing dependencies for the chart
  values.yaml         # The default configuration values for this chart
  charts/             # A directory containing any charts upon which this chart depends.
  templates/          # A directory of templates that, when combined with values,
                      # will generate valid Kubernetes manifest files.
  templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

| パス           | ファイル名           | 内容 | 必須 |
|----------------|-------------------|------|------|
| 適当なディレクトリ名/ |               |      |      |
|                | Chart.yaml        | Chartそのものの情報    | レ     |
|                | LICENSE           | ライセンス情報    |      |
|                | README.md         | README    |      |
|                | requirements.yaml | Chart間の依存関係を記述     |      |
|                | values.yaml       | チャートの変数のデフォルト値を記述    |  レ   |
|                | charts/           | チャートが依存しているチャートを格納している     |  レ    |
|                | templates/        | 有効なk8sのマニフェストファイルを出力するためのテンプレート     |  レ    |
|                | templates/NOTES.txt        | 端的に使い方を書いたテキストファイル     |      |

### デプロイ

今回はデモとして作成済みのRundeckの
チャートをインストールします。

#### helm install

```bash
$ helm install .
```

#### kubectlへのパイプ

```bash
$ helm template . | kubectl -f -
```

## 触ってみての感想

- チャートを作ること自体は大変
    - 手数自体は確実に増えるので作るか作らないかは要判断
- 標準で配布されているチャート
    - GKEなら`helm install`だけで動作するが、ローカル、オンプレだと動作しないもの多し
    - `README.md`をしっかり読んでチェックするべし
        - `helm fetch リポジトリ/チャート名`
        - `tar xvzf ダウンロードされたファイル.tgz`
        - 中にある`README.md`を確認
        - 環境にある程度合わせてvalueFileを作成
        - `helm install リポジトリ/チャート名 -f valueFile`

## まとめ(helmの活用ポイント)
