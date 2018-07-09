
# インストール先の確認

```
$ kubectl config current-context
docker-for-desktop
```

docker-for-desktopのコンテキストにインsトールされる。

# インストール時の注意点

admin権限を持ってるクラスタであれば特に配慮は不要。
RBACを有効かしている場合は注意。


# helmのインストール手順

https://github.com/kubernetes/helm/releases/
にアクセスして最新版をダウンロードする。

![](2018-07-05-14-08-58.png)

OSXなので OSXリンクを確認する。

今回は
```
https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-darwin-amd64.tar.gz
```
だったので、

```
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-darwin-amd64.tar.gz
$ tar xvzf helm-v2.9.1-darwin-amd64.tar.gz
x darwin-amd64/
x darwin-amd64/README.md
x darwin-amd64/helm
x darwin-amd64/LICENSE
$ cd darwin-amd64/
$ chmod +x helm
$ mv helm /usr/local/bin/
```

PATHが通っているかを確認する。

```
$ which helm
/usr/local/bin/helm
$ helm
The Kubernetes package manager

To begin working with Helm, run the 'helm init' command:

        $ helm init

This will install Tiller to your running Kubernetes cluster.
It will also set up any necessary local configuration.

Common actions from this point include:

- helm search:    search for charts
- helm fetch:     download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment:
  $HELM_HOME          set an alternative location for Helm files. By default, these are stored in ~/.helm
  $HELM_HOST          set an alternative Tiller host. The format is host:port
  $HELM_NO_PLUGINS    disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.
  $TILLER_NAMESPACE   set an alternative Tiller namespace (default "kube-system")
  $KUBECONFIG         set an alternative Kubernetes configuration file (default "~/.kube/config")

Usage:
  helm [command]

Available Commands:
  completion  Generate autocompletions script for the specified shell (bash or zsh)
  create      create a new chart with the given name
  delete      given a release name, delete the release from Kubernetes
  dependency  manage a chart's dependencies
  fetch       download a chart from a repository and (optionally) unpack it in local directory
  get         download a named release
  history     fetch release history
  home        displays the location of HELM_HOME
  init        initialize Helm on both client and server
  inspect     inspect a chart
  install     install a chart archive
  lint        examines a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      add, list, or remove Helm plugins
  repo        add, list, remove, update, and index chart repositories
  reset       uninstalls Tiller from a cluster
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  serve       start a local http web server
  status      displays the status of the named release
  template    locally render templates
  test        test a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client/server version information

Flags:
      --debug                           enable verbose output
  -h, --help                            help for helm
      --home string                     location of your Helm config. Overrides $HELM_HOME (default "/Users/01013548/.helm")
      --host string                     address of Tiller. Overrides $HELM_HOST
      --kube-context string             name of the kubeconfig context to use
      --tiller-connection-timeout int   the duration (in seconds) Helm will wait to establish a connection to tiller (default 300)
      --tiller-namespace string         namespace of Tiller (default "kube-system")

Use "helm [command] --help" for more information about a command.
```


# 参考

https://docs.helm.sh/using_helm/#quickstart-guide