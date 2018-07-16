<!-- TOC -->

- [PodとService](#pod%E3%81%A8service)
    - [Podの役割](#pod%E3%81%AE%E5%BD%B9%E5%89%B2)
        - [同一Podに含めるコンテナの例](#%E5%90%8C%E4%B8%80pod%E3%81%AB%E5%90%AB%E3%82%81%E3%82%8B%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%81%AE%E4%BE%8B)
        - [Podの定義例](#pod%E3%81%AE%E5%AE%9A%E7%BE%A9%E4%BE%8B)
            - [Service:Podの外部公開](#servicepod%E3%81%AE%E5%A4%96%E9%83%A8%E5%85%AC%E9%96%8B)

<!-- /TOC -->

# PodとService

## Podの役割

- 密に連携する複数のコンテナをまとめる

### 同一Podに含めるコンテナの例

1. Container AのファイルAが更新される
2. Container BのプロセスがファイルAの更新を検知
3. Container BのプロセスがファイルAの更新をコンテナ外部のサービスに通知する(or ファイルAをアップロードする)

このような関係の場合は同一Pod内部に含めることができる。

e.g. td-agentをサイドカーとして準備する

### Podの定義例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Nginxの最新イメージをPodとして定義する。

#### Service:Podの外部公開

通常の利用ではPodはクラスタ内部からのみしかアクセスされるできない。
したがって、Pod外部からアクセスできるようにServiceを定義する。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30000
```

```
[ external client ] ---------> [ nginx-service ] -------->  [ nginx ]
                       :30000       :80                        :80
```

Nodeの外側にはTCP 30000番でnginx-serviceのTCP80番ポートを公開する。
nginx-serviceはTCP80番で受信したパケットをnginxのTCP80番ポートに転送する。

**(注意)ここで述べているServiceの役割は本来のServiceの役割のほんの一部です**