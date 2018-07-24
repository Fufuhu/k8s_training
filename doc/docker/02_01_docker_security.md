# Dockerのセキュリティの確保について

Dockerの公式ドキュメントにDockerのセキュリティについて述べられたものがある。

[UNDERSTANDING DOCKER SECURITY AND BEST PRACTICES](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/)

これを自動的にチェックするためのツールが
**Docker Bench for Security**である。

[docker/docker-bench-security](https://github.com/docker/docker-bench-security)

これをここでは活用してよりセキュアな環境を実現させる。

## Docker Bench for Securityの実行

```bash
$ docker run -it --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security
```

これではパスしたものも大量に出てくるので警告(`WARN`)のみを洗い出す。

```bash
$ docker run -it --net host --pid host --userns host --cap-add audit_control \
    -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /usr/lib/systemd:/usr/lib/systemd \
    -v /etc:/etc --label docker_bench_security \
    docker/docker-bench-security | grep WARN
```

## Docker Bench for Securityの実行結果

単純にDockerをインストールした直後の状態を例示する。

```bash
$ docker run -it --net host --pid host --userns host --cap-add audit_control     -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST     -v /var/lib:/var/lib     -v /var/run/docker.sock:/var/run/docker.sock     -v /usr/lib/systemd:/usr/lib/systemd     -v /etc:/etc --label docker_bench_security     docker/docker-bench-security | grep WARN
[WARN] 1.1  - Ensure a separate partition for containers has been created
[WARN] 1.5  - Ensure auditing is configured for the Docker daemon
[WARN] 1.6  - Ensure auditing is configured for Docker files and directories - /var/lib/docker
[WARN] 1.7  - Ensure auditing is configured for Docker files and directories - /etc/docker
[WARN] 1.10 - Ensure auditing is configured for Docker files and directories - /etc/default/docker
[WARN] 2.1  - Ensure network traffic is restricted between containers on the default bridge
[WARN] 2.5  - Ensure aufs storage driver is not used
[WARN] 2.8  - Enable user namespace support
[WARN] 2.11 - Ensure that authorization for Docker client commands is enabled
[WARN] 2.12 - Ensure centralized and remote logging is configured
[WARN] 2.14 - Ensure live restore is Enabled
[WARN] 2.15 - Ensure Userland Proxy is Disabled
[WARN] 2.18 - Ensure containers are restricted from acquiring new privileges
[WARN] 4.5  - Ensure Content trust for Docker is Enabled
```

以降ではそれぞれのWARNについて警告内容への反映を行う。

### 1.1 - Ensure a separete partieion for containers has been created

```bash
$ sudo systemctl stop docker
```

```bash
$ sudo fdisk /dev/sdb
Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

コマンド (m でヘルプ): g

コマンド (m でヘルプ): n
パーティション番号 (1-128, default 1): 1
First sector (2048-209715166, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-209715166, default 209715166): 

Created a new partition 1 of type 'Linux filesystem' and of size 100 GiB.

コマンド (m でヘルプ): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```


```bash
$ sudo mkfs -t ext4 /dev/sdb1
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 26214139 4k blocks and 6553600 inodes
Filesystem UUID: 1f2c0bb1-967f-4d12-b304-e7c06f6de806
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```
`ext4`でフォーマットされたパーティション(dev/sdb1)が作成できたので、これを`/var/lib/docker`配下にマウントする。

`/etc/fstab`を修正して以下の内容を追加する。

```
/dev/sdb1	/var/lib/docker	ext4	defaults	0	0
```

```bash
$ sudo mount -a
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
~~~ 中略 ~~~
/dev/sdb1                           99G   60M   94G   1% /var/lib/docker
```

```
$ sudo systemctl start docker
```

`
[WARN] 1.1  - Ensure a separate partition for containers has been created
`の表示が消えていることを確認する。

### 1.5  - Ensure auditing is configured for the Docker daemon

```bash
$ sudo apt-get update
$ sudo apt-get install -y auditd
```
これはちょっとわからないので一旦スキップ

参考: https://github.com/nearform/devops/tree/master/packer/securing-docker

### 1.6  - Ensure auditing is configured for Docker files and directories - /var/lib/docker


```bash
# 1.5  - Ensure auditing is configured for the Docker daemon
echo "-w /usr/bin/docker -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.6  - Ensure auditing is configured for Docker files and directories - /var/lib/docker
echo "-w /var/lib/docker -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.7  - Ensure auditing is configured for Docker files and directories - /etc/docker"
echo "-w /etc/docker -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.8  - Ensure auditing is configured for Docker files and directories - docker.service
echo "-w /lib/systemd/system/docker.service -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.9  - Ensure auditing is configured for Docker files and directories - docker.socket
echo "-w /lib/systemd/system/docker.socket -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.10 - Ensure auditing is configured for Docker files and directories - /etc/default/docker
echo "-w /etc/default/docker -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.11 - Ensure auditing is configured for Docker files and directories - /etc/docker/daemon.json
echo "-w /etc/docker/daemon.json -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.12 - Ensure auditing is configured for Docker files and directories - /usr/bin/docker-containerd
echo "-w /usr/bin/docker-containerd -p wa" | sudo tee -a /etc/audit/audit.rules
# 1.13 - Ensure auditing is configured for Docker files and directories - /usr/bin/docker-runc
echo "-w /usr/bin/docker-runc -p wa" | sudo tee -a /etc/audit/audit.rules
```

```bash
$ sudo systemctl restart auditd
$ sudo auditctl -l
-w /usr/bin/docker -p wa
-w /var/lib/docker/ -p wa
-w /etc/docker/ -p wa
-w /lib/systemd/system/docker.service -p wa
-w /lib/systemd/system/docker.socket -p wa
-w /etc/default/docker -p wa
-w /etc/docker/daemon.json -p wa
-w /usr/bin/docker-containerd -p wa
-w /usr/bin/docker-runc -p wa
```

## 2.5  - Ensure aufs storage driver is not used

```bash
$ sudo docker info
〜〜〜〜中略〜〜〜〜
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
〜〜〜〜略〜〜〜〜
```

はい、残念。`aufs`を使っていました。
最新の`overlay2`ストレージドライバに変更しましょう。

[Dockerのストレージドライバの変更](https://qiita.com/RyoMa_0923/items/28d1aeb2d98b12fc7549) をベースに修正。


まず、dockerdを停止します。

```bash
$ sudo systemctl stop docker
```


`/lib/systemd/system/docker.service`を修正。

```text
ExecStart=/usr/bin/dockerd -H fd://
```

`ExecStart`に`--config-file /etc/docker/daemon.json`を追加します。

```text
ExecStart=/usr/bin/dockerd -H fd:// --config-file /etc/docker/daemon.json
```

```json
{
	"disable-legacy-registry": true,
	"icc": false,
	"live-restore": true,
	"storage-driver": "overlay2",
	"userland-proxy": false,
	"userns-remap": "default"
}
```

設定の変更を反映して、dockerdを再起動させます。

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start docker
```

`docker info`で利用しているストレージドライバを確認します。

```bash
$ docker info
〜〜〜〜中略〜〜〜〜
Storage Driver: overlay2
 Backing Filesystem: extfs
〜〜〜〜略〜〜〜〜
```

## [WARN] 2.12 - Ensure centralized and remote logging is configured


rsyslogdの設定(`/etc/rsyslog.conf`)を修正する。

```text
# provides TCP syslog reception
# module(load="imtcp")
# input(type="imtcp" port="514")
```
TCPでアクセスを受け付けるようにする。

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
	"disable-legacy-registry": true,
	"icc": false,
	"live-restore": true,
	"storage-driver": "overlay2",
	"log-driver": "syslog",
	"log-opts": {
    	"syslog-address": "tcp://127.0.0.1:514"
  	},
	"userland-proxy": false,
	"userns-remap": "default"
}
```

参考) https://www.nearform.com/blog/securing-docker-containers-on-aws/