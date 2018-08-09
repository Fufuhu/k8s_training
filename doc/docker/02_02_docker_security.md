Docker Bench for Securityを触った(2. Dockerデーモンの設定編)

[前回](https://qiita.com/RyoMa_0923/items/8537f0db6f6ba717cf39)に引き続き
[Docker Bench for Security](https://github.com/docker/docker-bench-security)を使って
dockerのセキュリティを改善を図ります。

今回の範囲は`Dockerデーモンの設定`です。



# 前提

Docker Bench for Securityを触った(1. Dockerホストの設定編)
と同じく以下の環境を利用します。

- Ubuntu 16.04
- docker 17.12.1-ce

# Docker Bench for Securityの実行結果(初期状態)

特に何も設定を行っていない状態では、以下のような結果となります。

```
[INFO] 2 - Docker daemon configuration
[WARN] 2.1  - Ensure network traffic is restricted between containers on the default bridge
[PASS] 2.2  - Ensure the logging level is set to 'info'
[PASS] 2.3  - Ensure Docker is allowed to make changes to iptables
[PASS] 2.4  - Ensure insecure registries are not used
[PASS] 2.5  - Ensure aufs storage driver is not used
[INFO] 2.6  - Ensure TLS authentication for Docker daemon is configured
[INFO]      * Docker daemon not listening on TCP
[INFO] 2.7  - Ensure the default ulimit is configured appropriately
[INFO]      * Default ulimit doesn't appear to be set
[WARN] 2.8  - Enable user namespace support
[PASS] 2.9  - Ensure the default cgroup usage has been confirmed
[PASS] 2.10 - Ensure base device size is not changed until needed
[WARN] 2.11 - Ensure that authorization for Docker client commands is enabled
[WARN] 2.12 - Ensure centralized and remote logging is configured
[INFO] 2.13 - Ensure operations on legacy registry (v1) are Disabled (Deprecated)
[WARN] 2.14 - Ensure live restore is Enabled
[WARN] 2.15 - Ensure Userland Proxy is Disabled
[PASS] 2.16 - Ensure daemon-wide custom seccomp profile is applied, if needed
[PASS] 2.17 - Ensure experimental features are avoided in production
[WARN] 2.18 - Ensure containers are restricted from acquiring new privileges
```

今回は上記のうち、`[WARN]`、`[INFO]`を対象として設定を行います。
ただし`2.6`についてはメッセージにあるとおり、TLSも何もTCPでリッスンしていないのでスキップします。




# 設定内容

## 2.1 - Ensure network traffic is restricted between containers on the default bridge

`/lib/systemd/system/docker.service`を修正。

```text
ExecStart=/usr/bin/dockerd -H fd://
```

`ExecStart`に`--config-file /etc/docker/daemon.json`を追加します。

```text
ExecStart=/usr/bin/dockerd -H fd:// --config-file=/etc/docker/daemon.json
```

`/etc/docker/daemon.json`を作成して、以下を追加します。

```json
{
	"icc": false
}
```

ここまで終わったら`dockerd`を再起動します。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

これによってコンテナの間をネットワーク接続させるには、`--link`オプションが必要になる点については注意してください。

## 2.5  - Ensure aufs storage driver is not used

最近のversionのdockerであれば`aufsは使っていないので警告は出ない`のでその点は注意してください。

```bash
$ sudo docker info
〜〜〜〜中略〜〜〜〜
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
〜〜〜〜略〜〜〜〜
```

はい、残念。`aufs`を使っていました。(現在のデフォルトではありません)
最新の`overlay2`ストレージドライバに変更しましょう。

[Dockerのストレージドライバの変更](https://qiita.com/RyoMa_0923/items/28d1aeb2d98b12fc7549) をベースに修正。


```json
{
	"icc": false,
	"storage-driver": "overlay2"
}
```

設定の変更が終わったら、dockerdを再起動させます。

```bash
$ sudo systemctl restart docker
```

`docker info`で利用しているストレージドライバを確認します。

```bash
$ docker info
〜〜〜〜中略〜〜〜〜
Storage Driver: overlay2 <------ここ
 Backing Filesystem: extfs
〜〜〜〜略〜〜〜〜
```

期待したとおりに`overlay2`になってますね。
ます。

## 2.7  - Ensure the default ulimit is configured appropriately

ここはちゃんと設定するようにここに書かれている設定はあくまでも
`[INFO]`を`[PASS]`にするための設定です。

実際には連想配列形式でここのulimitパラメータを記述してください。


```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true
}
```

## 2.8  - Enable user namespace support

実は最近のversionではすでにデフォルトで有効になってますが、明示的に有効化します。
`/etc/docker/daemon.json`に項目を追加します。

```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true,
	"userns-remap": "default"
}
```

## 2.12 - Ensure centralized and remote logging is configured

コンテナの出力するログをどこかに集約するようにします。
ここでは一番シンプルにログを集約できる`rsyslog`を利用します。

rsyslogdの設定(`/etc/rsyslog.conf`)を修正します。

```text
# provides TCP Init Binary: docker-init
syslog reception
# module(load="imtcp")
# input(type="imtcp" port="514")
```

rsystegがTCPでアクセスを受け付けるようにします。

```
# provides TCP syslog reception
module(load="imtcp")
input(type="imtcp" port="514")
$AllowedSender TCP, 127.0.0.1, 192.168.0.0/24
```

`log-driver`に`syslog`を指定し、
`log-opts`の`syslog-address`に`tcp://127.0.0.1:514`を指定。


```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true,
	"userns-remap": "default",
	"log-driver": "syslog",
	"log-opts": {
		"syslog-address": "tcp://127.0.0.1:514"
  	}
}
```

## 2.14 - Ensure live restore is Enabled

dockerデーモンが停止した場合もコンテナが停止しないようにします。
[Keep containers alive during daemon downtime](https://docs.docker.com/config/containers/live-restore/)

```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true,
	"userns-remap": "default",
	"log-driver": "syslog",
	"log-opts": {
		"syslog-address": "tcp://127.0.0.1:514"
  	},
	"live-restore": true
}
```

## 2.15 - Ensure Userland Proxy is Disabled

[ホスト上にコンテナのポートを割り当て](http://docs.docker.jp/engine/userguide/networking/default_network/binding.html)では、ヘアピンNATを有効にすることで、ユーザ空間のプロキシ(ループバックアドレスを利用)ではなく、iptablesのルールを使って通信する。

```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true,
	"userns-remap": "default",
	"log-driver": "syslog",
	"log-opts": {
    		"syslog-address": "tcp://127.0.0.1:514"
  	},
	"live-restore": true,
	"userland-proxy": false
}
```
## 2.18 - Ensure containers are restricted from acquiring new privileges

コンテナのプロセスに特権を追加できないようにする。
[Docker run リファレンス](http://docs.docker.jp/engine/reference/run.html)に
[記載の通り](https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt)、確認すると。**親プロセスからフォーク、クローンしたプロセスが新しい特権を付与できないことを保証する**らしいです。

```json
{
	"icc": false,
	"storage-driver": "overlay2",
	"default-ulimit": true,
	"userns-remap": "default",
	"log-driver": "syslog",
	"log-opts": {
    		"syslog-address": "tcp://127.0.0.1:514"
  	},
	"live-restore": true,
	"userland-proxy": false,
	"no-new-privileges": true
}
```

# まとめ

ここまででdockerデーモンの設定を通じてよりセキュアな
コンテナ実行環境を実現を図る方法を記載しました。

ただし、ここで行う設定については実際のdockerの使い勝手に影響する部分もあるので、
その点については配慮が必要そうです。