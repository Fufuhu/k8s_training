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

#### helm install

```bash
$ helm install . 
```

#### kubectlへのパイプ

```bash
$ helm template . | kubectl -f -
```

## 触ってみての感想

## helmの活用ポイント
