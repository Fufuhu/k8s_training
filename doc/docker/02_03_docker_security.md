Docker Bench for Securityを触った(3. Dockerデーモンの設定ファイル群編)

# 前提

Docker Bench for Securityを触った(1. Dockerホストの設定編)
と同じく以下の環境を利用します。

- Ubuntu 16.04
- docker 17.12.1-ce

# Docker Bench for Securityの実行結果

前回分(2. Dockerデーモンの設定編)まで実施したものであれば、
以下のような結果が得られます。

```
[INFO] 3 - Docker daemon configuration files
[PASS] 3.1  - Ensure that docker.service file ownership is set to root:root
[PASS] 3.2  - Ensure that docker.service file permissions are set to 644 or more restrictive
[PASS] 3.3  - Ensure that docker.socket file ownership is set to root:root
[PASS] 3.4  - Ensure that docker.socket file permissions are set to 644 or more restrictive
[PASS] 3.5  - Ensure that /etc/docker directory ownership is set to root:root
[PASS] 3.6  - Ensure that /etc/docker directory permissions are set to 755 or more restrictive
[INFO] 3.7  - Ensure that registry certificate file ownership is set to root:root
[INFO]      * Directory not found
[INFO] 3.8  - Ensure that registry certificate file permissions are set to 444 or more restrictive
[INFO]      * Directory not found
[INFO] 3.9  - Ensure that TLS CA certificate file ownership is set to root:root
[INFO]      * No TLS CA certificate found
[INFO] 3.10 - Ensure that TLS CA certificate file permissions are set to 444 or more restrictive
[INFO]      * No TLS CA certificate found
[INFO] 3.11 - Ensure that Docker server certificate file ownership is set to root:root
[INFO]      * No TLS Server certificate found
[INFO] 3.12 - Ensure that Docker server certificate file permissions are set to 444 or more restrictive
[INFO]      * No TLS Server certificate found
[INFO] 3.13 - Ensure that Docker server certificate key file ownership is set to root:root
[INFO]      * No TLS Key found
[INFO] 3.14 - Ensure that Docker server certificate key file permissions are set to 400
[INFO]      * No TLS Key found
[PASS] 3.15 - Ensure that Docker socket file ownership is set to root:docker
[PASS] 3.16 - Ensure that Docker socket file permissions are set to 660 or more restrictive
[PASS] 3.17 - Ensure that daemon.json file ownership is set to root:root
[PASS] 3.18 - Ensure that daemon.json file permissions are set to 644 or more restrictive
[PASS] 3.19 - Ensure that /etc/default/docker file ownership is set to root:root
[PASS] 3.20 - Ensure that /etc/default/docker file permissions are set to 644 or more restrictive4 or more restrictive
```

今回は上記のうち、`[WARN]`、`[INFO]`を対象として設定を行います。

# 設定内容

## 3.7  - Ensure that registry certificate file ownership is set to root:root

dockerクライアントとdockerデーモン間の通信をTLSを使って暗号化する場合に利用します。
詳細については、[Protect the Docker daemon socket](https://docs.docker.com/engine/security/https/)を参照してください。

ここでは、これらの設定を適用後に設定ファイルに正しい所有権限を付与するための手順について解説します。

基本的には証明書を配置するためのディレクトリを作成します。

```bash
$ sudo mkdir /etc/docker/cert.d
$ sudo ls -lah /etc/docker/cert.d
合計 16K
drwx------  3 root root 4.0K  8月 10 01:36 .
drwxr-xr-x 92 root root 4.0K  8月  3 06:12 ..
drwxr-xr-x  2 root root 4.0K  8月 10 01:27 certs.d
```

## 3.8  - Ensure that registry certificate file permissions are set to 444 or more restrictive

証明書ファイル(.crtファイル)のパーミッションが444を適用します。

下のイメージ(?)は[Verify repository client with certificates](https://docs.docker.com/engine/security/certificates/#creating-the-client-certificates)
から抜粋したものです。

```
    /etc/docker/certs.d/        <-- Certificate directory
    └── localhost:5000          <-- Hostname:port
       ├── client.cert          <-- Client certificate
       ├── client.key           <-- Client key
       └── ca.crt               <-- Certificate authority that signed
                                    the registry certificate
```

この場合は、
`ca.crt`ファイルのパーミッションが`444`、またはより厳しくなっていれば`PASS`になります。

```
$ sudo chmod 444 /etc/docker/certs.d/localhost:5000/ca.crt
```

## 3.9  - Ensure that TLS CA certificate file ownership is set to root:root
## 3.10 - Ensure that TLS CA certificate file permissions are set to 444 or more restrictive

3.9、3.10では、dockerdのオプションに`--tlscacert`が指定されており、
そこで指定されているファイルの所有権限が`root:root`に設定されているかつ、
パーミッションが`444`(またはより厳しい)に設定されていれば`PASS`となります。

たとえば、`--tlscacert=/etc/docker/ca.pem`が指定されている場合は、以下のように設定します。

```bash
$ sudo chmod 444 /etc/docker/ca.pem
$ sudo chown root:root /etc/docker/ca.pem
```

**そもそも、`--tlscacert`が指定されていない場合は関係ない**点には留意しましょう。

## 3.11 - Ensure that Docker server certificate file ownership is set to root:root
## 3.12 - Ensure that Docker server certificate file permissions are set to 444 or more restrictive

3.11, 3.12ではdockerdのオプションに`--tlscert`が指定されており、
そこで指定されているファイルの所有権限が`root:root`に指定されているかつ、
パーミッションが`444`(またはより厳しい)に設定されていれば`PASS`となります。

たとえば、`--tlscert=/etc/docker/cert.pem`が指定されている場合は、以下のように設定します。

```bash
$ sudo chmod 444 /etc/docker/cert.pem
$ sudo chown root:root /etc/docker/cert.pem
```

## 3.13 - Ensure that Docker server certificate key file ownership is set to root:root
## 3.14 - Ensure that Docker server certificate key file permissions are set to 400

3.9-3.12までと同じく、dockerdのオプションに`--tlskey`が指定されており、
そこで指定されているファイルの所有権限が`root:root`に指定されているかつ、
パーミッションが`400`に設定されていれば`PASS`となります。

たとえば、`--tlskey=/etc/docker/key.pem`が指定されている場合は、以下のように設定します。

```bash
$ sudo chmod 400 /etc/docker/key.pem
$ sudo chown root:root /etc/docker/key.pem
```

# まとめ

今回はDockerデーモンの各種設定ファイルについて、セキュリティを改善するための手順について解説しました。
実質的に証明書周りが適切に設定されているかがほとんどでした。