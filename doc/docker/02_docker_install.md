# Dockerのインストール

## 概要

Rancher Labs社のリポジトリにインストール用の
シェルスクリプトが公開されているのでこれを利用する。

https://github.com/rancher/install-docker

任意のバージョンのDockerがインストール可能だが、
今回は諸事情によりv17.03をインストールする。

## 手順詳細

### Dockerのインストール

```bash
$ curl https://releases.rancher.com/install-docker/17.03.sh | bash
```

### sudoなしでコマンドを実行可能にする

sudoなしでdockerコマンドを実行できるように、
自分自身をdockerグループに所属させる。

```bash
$ sudo usermod -aG docker fujiwara
```

`docker`コマンドが正常に実行できればOK。

```bash
$ $ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

