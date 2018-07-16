# クラスタの構築順

## SELinuxの無効化

いったん無効化

```
$ sudo apt-get update
$ sudo apt-get install selinux-basics selinux-policy-default auditd
```

## dockerのインストール

### 古いバージョンのdockerの削除

```
$ sudo apt-get remove docker-ce
```

### 新しいバージョンのdockerをインストール

https://docs.docker.com/install/linux/docker-ce/ubuntu/

の手順に従って新しいverのdockerをインストールする



## rkeのダウンロード

## パスなしsshの有効化
https://www.server-world.info/query?os=Ubuntu_16.04&p=ssh&f=4

https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7


## rke設定ファイルの作成

`rke config`コマンドのウィザードに答える形でrkeの設定ファイルを作成可能。

```
$ rke config
[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]: 
[+] Number of Hosts [1]: 
[+] SSH Address of host (1) [none]: 127.0.0.1
[+] SSH Port of host (1) [22]: 
[+] SSH Private Key Path of host (127.0.0.1) [none]: 
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (127.0.0.1) [none]: 
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (127.0.0.1) [ubuntu]: fujiwara
[+] Is host (127.0.0.1) a control host (y/n)? [y]: y
[+] Is host (127.0.0.1) a worker host (y/n)? [n]: y
[+] Is host (127.0.0.1) an Etcd host (y/n)? [n]: y
[+] Override Hostname of host (127.0.0.1) [none]: kubernetes
[+] Internal IP of host (127.0.0.1) [none]: 
[+] Docker socket path on host (127.0.0.1) [/var/run/docker.sock]: 
[+] Network Plugin Type (flannel, calico, weave, canal) [canal]: 
[+] Authentication Strategy [x509]: 
[+] Authorization Mode (rbac, none) [rbac]: 
[+] Kubernetes Docker image [rancher/hyperkube:v1.10.5-rancher1]: 
[+] Cluster domain [cluster.local]: 
[+] Service Cluster IP Range [10.43.0.0/16]: 
[+] Enable PodSecurityPolicy [n]: y
[+] Cluster Network CIDR [10.42.0.0/16]: 
[+] Cluster DNS Service IP [10.43.0.10]: 
[+] Add addon manifest urls or yaml files [no]:
```

```
$ cat cluster.yml 
```

```yaml
# If you intened to deploy Kubernetes in an air-gapped environment,
# please consult the documentation on how to configure custom RKE images.
nodes:
- address: 127.0.0.1
  port: "22"
  internal_address: ""
  role:
  - controlplane
  - worker
  - etcd
  hostname_override: kubernetes
  user: fujiwara
  docker_socket: /var/run/docker.sock
  ssh_key: ""
  ssh_key_path: ~/.ssh/id_rsa
  labels: {}
services:
  etcd:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    external_urls: []
    ca_cert: ""
    cert: ""
    key: ""
    path: ""
    snapshot: false
    retention: ""
    creation: ""
  kube-api:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    service_cluster_ip_range: 10.43.0.0/16
    service_node_port_range: ""
    pod_security_policy: true
  kube-controller:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    cluster_cidr: 10.42.0.0/16
    service_cluster_ip_range: 10.43.0.0/16
  scheduler:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
  kubelet:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
    cluster_domain: cluster.local
    infra_container_image: ""
    cluster_dns_server: 10.43.0.10
    fail_swap_on: false
  kubeproxy:
    image: ""
    extra_args: {}
    extra_binds: []
    extra_env: []
network:
  plugin: canal
  options: {}
authentication:
  strategy: x509
  options: {}
  sans: []
addons: ""
addons_include: []
system_images:
  etcd: rancher/coreos-etcd:v3.1.12
  alpine: rancher/rke-tools:v0.1.10
  nginx_proxy: rancher/rke-tools:v0.1.10
  cert_downloader: rancher/rke-tools:v0.1.10
  kubernetes_services_sidecar: rancher/rke-tools:v0.1.10
  kubedns: rancher/k8s-dns-kube-dns-amd64:1.14.8
  dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.14.8
  kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.14.8
  kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0
  kubernetes: rancher/hyperkube:v1.10.5-rancher1
  flannel: rancher/coreos-flannel:v0.9.1
  flannel_cni: rancher/coreos-flannel-cni:v0.2.0
  calico_node: rancher/calico-node:v3.1.1
  calico_cni: rancher/calico-cni:v3.1.1
  calico_controllers: ""
  calico_ctl: rancher/calico-ctl:v2.0.0
  canal_node: rancher/calico-node:v3.1.1
  canal_cni: rancher/calico-cni:v3.1.1
  canal_flannel: rancher/coreos-flannel:v0.9.1
  wave_node: weaveworks/weave-kube:2.1.2
  weave_cni: weaveworks/weave-npc:2.1.2
  pod_infra_container: rancher/pause-amd64:3.1
  ingress: rancher/nginx-ingress-controller:0.10.2-rancher3
  ingress_backend: rancher/nginx-ingress-controller-defaultbackend:1.4
ssh_key_path: ~/.ssh/id_rsa
ssh_agent_auth: false
authorization:
  mode: rbac
  options: {}
ignore_docker_version: false
kubernetes_version: ""
private_registries: []
ingress:
  provider: ""
  options: {}
  node_selector: {}
  extra_args: {}
cluster_name: ""
cloud_provider:
  name: ""
prefix_path: ""
addon_job_timeout: 0
bastion_host:
  address: ""
  port: ""
  user: ""
  ssh_key: ""
  ssh_key_path: ""
```

## clusterの構築

```
$ rke up --ignore-docker-version
```

## kubectlのインストール

https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-snap-on-ubuntu

の手順に従う。

```
$ sudo snap install kubectl --classic
```


### 動作確認

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80

```


なんかRKEじゃ無理くさいので

https://linuxconfig.org/how-to-install-kubernetes-on-ubuntu-18-04-bionic-beaver-linux

にしたがって準備することにしよう

```
sudo apt install docker.io
sudo systemctl unmask docker.service
sudo usermod -aG docker fujiwara
sudo systemctl restart docker
```

dockerグループが存在しない場合は
```
sudo groupadd docker
```
を実行した上で`usermod`を実行する。