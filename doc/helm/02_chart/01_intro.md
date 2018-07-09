
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