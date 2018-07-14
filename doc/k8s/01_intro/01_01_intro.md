# Kubernetesの紹介

## 目的

1. Kubernetesとは何かを理解する
2. Kubernetesの活用のメリット・デメリットを理解する

この2点についてこれまでにパブリックに出てきた概念などを交えながら説明する。

## 前提

Kubernetesではcontainerd, rkt, gVisorなどもコンテナランタイムとして
利用することが現在では可能ですが、2018年7月時点で最もポピュラーで
あると思われるDockerをベースとして利用する前提で記述します。

## Kubernetesとは何か

コンテナ環境のオーケストレーションツールです。
実態は、分散環境におけるOSという風に認識したほうが今では正確かもしれません。

Cloud Native Trail Mapをベースに考えてみる。
2018年7月8日時点では、Cloud Native Trail Mapの定義する7ステップのうち、
3ステップ目に位置します。

1. Containerization
2. CI/CD
3. Orchestration & Application Definition

## Containerization

アプリケーションをコンテナの形で利用できるようにします。
基本的には、以下のことに留意してコンテナ化します。

1. 1コンポーネント = 1コンテナ
2. 12 Factor Appの原則に準拠させる

基本的な原則はこのとおりですが、
現時点でのコンテナ環境およびk8s環境の制約事項として、
DBについてはコンテナ化しないほうがおすすめです。(ただしCI時のテストなどは除く)

12 Factor Appについては補足資料([01_02_12_Factor.md](./01_02_12_Factor.md))を
参照してください。

## CI/CD

12 Factor Appに可能な限り準拠する形でアプリケーションを構成したらCI/CDに移ります。
特に12 Factor Appでこの時点で意識すべきは、以下の項目です。

- 開発・本番一致

完全に一致させることは無理でも可能な限り近づけることを目指します。

これは、
1. 各種設定項目
2. インテグレーションおよびデプロイ手順
の両方をまったく同じ手順で実行できることを目指します。

特に、2については開発と本番でシステム的な作業としては同一の方法で実施できるようにします。

またここで定めたCIにてテストを通過したもののみをデプロイ可能にできるようにすることが
システム全体の秩序を維持することにつながります。

## Orchestration & Application Definition

Kubernetes(とHelm)が属するのがこのステップになります。

KubernetesはおもにOrchestration(とApplication Definitionの一部)を担います。
HelmはApplication Definitionを担います。

### Orchestration

クラウドコンピューティングなどと異なり、
オーケストレーションには明確な定義はありませんが
主に以下を解決するための手段を表します。

- 複数のDockerホストをどう管理するか
- どのDockerホストにコンテナ（アプリケーション）を立ち上げるか
- コンテナのデプロイ方法をどう定義するか
- 複数のコンテナ間の連携をどう実現するか
- コンテナの死活監視をどうするか
- コンテナの保持するデータをどう管理するか
- 外部ネットワークからコンテナへのアクセス経路をどう設定するか




### 補足 Helmについて

Helmはk8sにおけるパッケージマネージャです。

12 Factor Appsにおける以下の項目を担います。

- コードベース
- 依存関係
- 設定

具体的な使い方や機能については別の機会で説明します。

# 参考文献
[先行事例に学ぶKubernetes活用企業の実態](http://www.atmarkit.co.jp/ait/series/9283/)
[Cloud Native trail map](https://github.com/cncf/landscape/tree/master/trail_map)
[The Twelve-Factor App 日本語訳版](https://12factor.net/ja/)