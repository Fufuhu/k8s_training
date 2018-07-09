
# チャートファイルの構造

```
wordpress/
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
これらが予約語的に運用される点については注意してください。


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


## README.mdとLICENSE、templates/NOTES.txt

LICENSEは文字通り
README.mdで詳細なマニュアルを提供する
templates/NOTES.txtで具体的なマニュアルを提供する。



# 個別のファイルの解説

## Chart.yml

```
apiVersion: The chart API version, always "v1" (required)
name: The name of the chart (required)
version: A SemVer 2 version (required)
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this project's home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
maintainers: # (optional)
  - name: The maintainer's name (required for each maintainer)
    email: The maintainer's email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
engine: gotpl # The name of the template engine (optional, defaults to gotpl)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). This needn't be SemVer.
deprecated: Whether this chart is deprecated (optional, boolean)
tillerVersion: The version of Tiller that this chart requires. This should be expressed as a SemVer range: ">2.0.0" (optional)
```

| 項目          | 必須 | 内容                                 |
|---------------|------|--------------------------------------|
| apiVersion    | レ    | チャートのAPIバージョン(v1固定)               |
| name          | レ    | チャートの名前                            |
| version       | レ    | セマンティックバージョニング(後述)                 |
| kubeVersion   |      | 互換性のあるk8sのバージョン                   |
| description   |     | プロジェクトの端的な説明                     |
| keywords      |      | このプロジェクトに対するキーワードのリスト(リスト形式で記述) |
| home          |      | このプロジェクトのホームページURL                   |
| sources       |      | このプロジェクトのURLソースコード                   |
| maintaners    |     | メンテナのリスト(記法は後述)                  |
| engine        |      | gotpl                                |
| icon          |      | SVGまたはPNGアイコンのURL                    |
| appVersion    |      | アプリケーションのバージョン                       |
| deprecated    |      | Deprecatedか否かのフラグ                   |
| tillerVersion |      | 必要なTillerのバージョン                    |

## versionの部分

nginxのチャートでversionに1.2.3を指定した場合は生成されるパッケージは以下のようになる。

```
nginx-1.2.3.tgz
```

# Chartの依存関係


```
[Chart A] <-----┰----[Chart B]
                ┣----[Chart C]
                ┗----[Chart D]
```

チャートの依存関係は
`requirements.yaml`または、`charts/`ディレクトリを介して
動的に接続される。

- 依存関係の記述
  - requirements.yaml
  - chartsディレクトリ
    - chartsディレクトリ配下にチャートを配置する?

## requirements.yamlの書き方

```
dependencies:
  - name: apache
    version: 1.2.3
    repository: http://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: http://another.example.com/charts
```

|フィールド名|内容|
|---|---|
|name|チャートの名前|
|version|チャートのバージョン|
|repository|チャートリポジトリのURL|

repositoryを指定する際には、`helm repo add`を実行しなければいけない。

### helm dependency update

依存関係情報を記述したら、`helm dependency update チャート名`で
chart/ディレクトリ配下に依存関係に記述したチャートをダウンロードできる。

先に記載した依存関係のファイルの場合、
`charts/`ディレクトリ配下は以下のようになる。

```
charts/
 ┣-apache-1.2.3.tgz
 ┗-mysql-3.2.1tgz
```

requirements.yamlをつかって依存間駅を管理するのはチャートを
アップデートされた状態に保つのに良い。

### aliasフィールド

```
# parentchart/requirements.yaml
dependencies:
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-1
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
    alias: new-subchart-2
  - name: subchart
    repository: http://localhost:10191
    version: 0.1.0
```

この場合、subchartに対して、new-subchart-1, new-subchart-2の別名(alias)をつけている。
同じ名前のチャートが重複した場合に使うと良い。

### tagsとconditionフィールド

デフォルトでは、依存関係のある全てのチャートが読み込まれるが、
`tags`と`condition`フィールドを使うことである程度の制御が可能である。

- Condition
  - カンマ区切りのYAMLパスを保つ。
  - パスが存在する場合は最上位のペアレントではbooleanとしてTrueになる。
  - booleanの値に応じてチャートの有効・無効が変更される。
- Tags
  - チャートに紐づくYAMLのリスト。
  - 最上位のペアレントでタグの有無でチャートの有効、無効が切り替えられる

```
[parentchart] <--┰--[subchart1]
                 ┃
                 ┗--[subchart2]
```

```
# 親チャートの requirements.yaml
dependencies:
      - name: subchart1
        repository: http://localhost:10191
        version: 0.1.0
        condition: subchart1.enabled,global.subchart1.enabled
        tags:
          - front-end
          - subchart1

      - name: subchart2
        repository: http://localhost:10191
        version: 0.1.0
        condition: subchart2.enabled,global.subchart2.enabled
        tags:
          - back-end
          - subchart2
```


```
# 親チャートのvalues.yaml
subchart1:
  enabled: true
tags:
  front-end: false
  back-end: true
```

この場合は、values.yamlの指定がなければ全てのチャートが無効になります。
```
tags:
  front-end: false
```
の記述部分に起因する。

一方で

```
subchchart1:
  enabled: true
```

となっているので、


values.yaml内部の指定により、`front-end`タグをもつ全てのチャートは`subchart1.enabled`パス
が`true`判定になるので、`front-end`タグが~~~~よくわからん。。。。

yamlファイルではなく、コマンドライン上で値を渡すことでも可能
(あくまでもテスト・デバッグ向け)

```
helm install --set tags.front-end=true --set subchart2.enabled=false
```

#### TagとConditionの解決

1. ConditionはTagの指定に優先される
2. Tagはチャートのいずれかが真であればチャートを有効化する。(全てではなく、どれかがTrueなら有効になる)
3. TagとConditionはトップに配置されないといけない

#### 子チャートの値を親チャートに伝播させる

子チャートの値を、共通のデフォルトとして親チャートに伝播させた方が良い時がある。

- `exports`フォーマットのもう一つの利点
  - ユーザが設定可能な値をチェックすることが可能になる。

```
[Parent's requirements.yaml] <----- [Child's values.yaml]
```

```
# parent's requirements.yaml file
    ...
    import-values:
      - data
```

```
# child's values.yaml file
...
exports:
  data:
    myint: 99
```

このように親チャートのrequirements.yamlで子チャートのdataをimportするようにすることで、
子チャートのvalues.yamlに記述されたdataの値を受け取ることができる。

この場合、親チャートのvalues.yamlは以下が追加される。

```
...
myint: 99
...
```

`data`が親チャートに映る際に消える点には注意すること。

```
[export: @child's values] -> [import-values: @parent's requirements.yaml] -> [@parent's values.yaml]
```

## charts/ディレクトリを使った手動管理

charts/ディレクトリ配下にchart設定を置くことでも依存関係を管理できる。
基本的には、より詳細に依存関係を管理したい場合に利用する。

## 依存関係を利用することについての運用観点整理

```
A名前空間
  ┣ Aステートフルセット
  ┗ Aサービス
```

```
B名前空間
  ┣ Bレプリカセット
  ┗ Bサービス
```

AがBに依存している場合、以下の順番でリソースが作成される。

1. A名前空間
2. B名前空間
3. Aステートフルセット
4. Bレプリカセット
5. Aサービス
6. Bサービス

単一のリリースが作成されたらチャートのすべてのオブジェクトとその依存関係がオブジェクトが作成される。

# テンプレートと値

すべてのテンプレートファイルは `templates/` ディレクトリに格納される。

- テンプレートに対する値の導入方法
  - `values.yaml` ファイルの利用(デフォルト値の提供)
  - `helm install` 時にファイルを指定

## テンプレートファイルの書き方

[Go templates](https://golang.org/pkg/text/template/)の書き方に準じる。

## values.yamlの扱い

`values.yaml`ファイルは最小限必要なデフォルト値の提供を機能として担う。

### 他のyamlファイルによるオーバーライド

`helm install --values=独自の値のyamlファイル.yaml チャート名`

とすると、values.yamlの値は上書きされる。

### install/upgrade時のオプション(--set)

- `helm install --set A=B チャート名`
- `halm upgrade --set A=B リリース名`

などとすることで上書きできるが、**コマンド形式はfreakyな状況を招く**ので、利用は進めない。

## テンプレートのファイルの例と解説

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    heritage: deis
spec:
  replicas: 1
  selector:
    app: deis-database
  template:
    metadata:
      labels:
        app: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{.Values.imageRegistry}}/postgres:{{.Values.dockerTag}}
          imagePullPolicy: {{.Values.pullPolicy}}
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{default "minio" .Values.storage}}
```

### valuesファイル

`{{ .Values.キー名 }}` で値を提供するファイルからの値を読み込むことができる。

### 事前定義済みのキー

|キー|内容|備考|
|---|---|---|
|Release.Name|リリースの名前||
|Release.Time|チャートのリリースが最後にアップデートされた時間||
|Release.Namespace|チャートが紐づいているk8sの名前空間||
|Release.Service|リリースを実行しているサービス、通常は`Tiller`になる||
|Release.IsUpgrade|Upgrade/Rollbackか否か||
|Release.IsInstall|Installか否か||
|Release.Revision|`helm upgrade`を実行するたびに値が上がる(1始まり)||
|Chart|`Chart.yaml`の中身。`Chart.Version`などでChart.yamlに記述された値を取り出すことができる||
|Files|チャートに含まれる全ての特別でないファイルを含んだマップ様オブジェクト|詳細は別途調査&説明|
|Capabilities|k8sのバージョンについての情報を含んだマップ様オブジェクト||

#### Capabilitiesについて

チャートおよびリリースがどのような環境であれば実行可能であるかの情報があらわれる。

|キー|内容|
|---|---|
|.Capabilities.KubeVersion|対応するk8sのバージョン|
|.Capabilities.TillerVersion|対応するTillerのバージョン|
|.Capabilitik.Has "batch/v1"|batch/v1を持つAPIバージョン|

## スコープ、依存とValues

Valuesは一番上位のチャートでのみ宣言できる。

valuesファイルは、その依存関係内部にも値を提供することができる。

下の例はwordpressチャートに含まれるmysqlチャートとapacheチャートに
値を提供している。

```
title: "My WordPress Site" # Sent to the WordPress template

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

この場合、MySQLは `.Values.mysql.password`にはアクセスできるが、
`title`にはアクセスできない。


値は名前空間で区切られているのが、名前空間情報は取り除くことができる。
Wordpressチャートからは`.Values.mysql.password`でMySQLのパスワードにアクセスするが、
MySQLチャートでは`mysql`名前空間内部なこともあり、`.Values.password`でMySQLのパスワードにアクセスできる。

### Globalスコープ

```
title: "My WordPress Site" # Sent to the WordPress template

global:
  app: MyWordPress

mysql:
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  port: 8080 # Passed to Apache
```

チャートからはトップレベルでも、依存関係に含まれるものでも `{{ .Values.global.app }}`でアクセスできる。

```
title: "My WordPress Site" # Sent to the WordPress template

global:
  app: MyWordPress

mysql:
  global:
    app: MyWordPress
  max_connections: 100 # Sent to MySQL
  password: "secret"

apache:
  global:
    app: MyWordPress
  port: 8080 # Passed to Apache
```

内部的には、こう解釈される。

# チャートの作成・パッケージング・テスト

- 作成
  - `helm create チャート名`
- パッケージング
  - `helm package チャート名`
- テスト(Lint)
  - `helm lint チャート名`


# チャートリポジトリ

- チャートリポジトリ
  - チャートのパッケージを格納するHTTPサーバ
  - `helm`自体のローカルリポジトリを`helm serve`でチャートリポジトリとして提供することも可能(テスト・開発用)
  - YAMLファイルを提供できるHTTPサーバであればなんでもOK

- `index.yaml`にリポジトリないの全チャートの情報(メタデータ含む)が含まれていればOK
- クライアントサイドでは`helm repo`で色々できる`


## チャートのスターターパック

`helm create --startar`でチャートのベースとなるファイル一式を`$HELM_HOME/starters`ディレクトリ配下に存在する。

# Hook

Helmは処理の途中で割り込みを行なって色々な動作を実現できるようにしている。

- 利用例
  1. Chart読み込みの前にConfigMapやSecretを読み込む
  2. 新しいチャートをインストールする前にDBをバックアップするジョブを実行して、データをアップグレードする
  3. サービスをローテションの外にグレースフルに持っていくジョブを特定のリリースを削除する前に実行する


## Hook一覧

1. pre-install
2. post-install
3. pre-delete
4. post-delete
5. pre-upgrade
6. post-upgrade
7. pre-rollback
8. post-rollback
9. crd-install

### Hookとリリースのライフサイクル

1. `helm install foo`
2. Tillerにチャートが読み込まれる
3. バリデーションの後にfooのテンプレートがレンダリングされる
4. Tillerがk8sに最終的に出来上がったリソースを読み込ませる
5. Tillerがリリースの名前をクライアントに返す
6. クライアントが終了

### Hookの例

- Hookもk8sのリソースとして記述する。
- `annotations`でどのタイミングをHookとして実行するかを指定

```
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{.Release.Name}}"
  labels:
    heritage: {{.Release.Service | quote }}
    release: {{.Release.Name | quote }}
    chart: "{{.Chart.Name}}-{{.Chart.Version}}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{.Release.Name}}"
      labels:
        heritage: {{.Release.Service | quote }}
        release: {{.Release.Name | quote }}
        chart: "{{.Chart.Name}}-{{.Chart.Version}}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{default "10" .Values.sleepyTime}}"]
```

この例の場合は`post-install`で実行する。
もう少し細かく書いせる

```
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install # post-installで実行
    "helm.sh/hook-weight": "-5" # 同一hook設定が存在した場合の優先度を指定(-5なのでかなり優先度高め)
    "helm.sh/hook-delete-policy": hook-succeeded # hookが成功したら削除する
```

# テンプレートのTips

## Imageのpullシークレット

```
imageCredentials:
  registry: quay.io
  username: someone
  password: sillyness
```

```
{{- define "imagePullSecret" }}
{{- printf "{\"auths\": {\"%s\": {\"auth\": \"%s\"}}}" .Values.imageCredentials.registry (printf "%s:%s" .Values.imageCredentials.username .Values.imageCredentials.password | b64enc) | b64enc }}
{{- end }}
```

```
apiVersion: v1
kind: Secret
metadata:
  name: myregistrykey
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "imagePullSecret" . }}
```

こんな感じ。

## ConfigMap/Secretsの変更でdeploymentsをrolling upgradeする

```
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
[...]
```

`annotations`で変更を検知する対象を指定した後に、
`helm upgrade`を実行することで変更できる。
(ただし、ConfigMapやSecretsの変更が実際に発生した場合のみ)

## PARTIALとテンプレートのInclude

templateディレクトリ配下の`_`で始まるファイルは
k8sのマニフェストファイルを直接出力されることは期待されない。
慣習的に、`_helpers.tpl`ファイルに部分が記述される

