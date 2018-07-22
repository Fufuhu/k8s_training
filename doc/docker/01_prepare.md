# 事前準備

Dockerをインストールする事前準備を行う。

## 前提

- Ubuntu 16.04


## 手順

`/etc/network/interfaces`を修正

```text
# The primary network interface
auto ens160
iface ens160 inet dhcp
```

これを

```text
# The primary network interface
auto ens160
    iface ens160 inet static
    address 192.168.0.100
    netmask 255.255.255.0
    gateway 192.168.0.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

に修正。

```
$ sudo systemctl restart networking
$ sudo reboot
```

同様にもう一台のマシンも`192.168.0.101`で固定。

## 公開鍵でログインできるようにする

事前に先程作成したサーバのホームディレクトリに`.ssh`ディレクトリを作成しておく。

```text
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/fujiwara/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/fujiwara/.ssh/id_rsa.
Your public key has been saved in /home/fujiwara/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:rsgp3wEcyhfx1Owd9MCpMkBNmUxBvcxA8B0rg5CIKK0 fujiwara@kubernetes02
The key's randomart image is:
+---[RSA 2048]----+
|oo.o+OBB.oo.     |
|= o..**.+o+o     |
|..  +.==o+ ..    |
|E. o ooo* .      |
|  o +  oS        |
|   . . .         |
|      . .        |
|  .. + o         |
|   o= o          |
+----[SHA256]-----+
```

```text
$ scp authorized_keysfujiwara@192.168.0.100:/home/fujiwara/.ssh/authorized_keys
(中略)
$ scp authorized_keysfujiwara@192.168.0.101:/home/fujiwara/.ssh/authorized_keys
```

パスワードなしてログインできることを確認する。

```text
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

231 個のパッケージがアップデート可能です。
137 個のアップデートはセキュリティアップデートです。


Last login: Sat Jul 21 02:54:30 2018 from 192.168.0.6
```


## パスなしsudoの設定

`/etc/sudoers.d/ユーザ名`を作成して以下のように入力する。

```text
ユーザ名    ALL=(ALL) NOPASSWD:ALL
```

例えばこんな感じ。

```text
$ cat /etc/sudoers.d/fujiwara
fujiwara    ALL=(ALL) NOPASSWD:ALL
```