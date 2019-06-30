+++
title = "Pods"
subtitle = "Kubernetes pods by example"
date = "2019-02-27"
url = "/pods/"
+++

pod はネットワークと名前空間を共有するコンテナの集まりで、Kubernetes におけるデプロイメントの基本単位です。pod 内のコンテナはすべて同じ node 上でスケジュールされます。

コンテナの[イメージ](https://hub.docker.com/r/mhausenblas/simpleservice/) `mhausenblas/simpleservice:0.5.0` を使って、HTTP API をポート番号 `9876` で公開する pod を起動するには、以下を実行します。

```bash
$ kubectl run sise --image=mhausenblas/simpleservice:0.5.0 --port=9876
```

pod が実行中であることは次のように確認できます。

```bash
$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
sise-3210265840-k705b     1/1       Running   0          1m

$ kubectl describe pod sise-3210265840-k705b | grep IP:
IP:                     172.17.0.3
```

クラスタ内部から (たとえば `minishift ssh` を使うと) この pod には pod IP `172.17.0.3` でアクセスできます。IP アドレスは上のコマンド `kubectl describe` からわかります。

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.5.0", "from": "172.17.0.1"}
```

なお、`kubectl run` は [deployment](/deployments/) を作成します。そのため、作成した pod を削除するには `kubectl delete deployment sise` を実行してください。


#### 設定ファイルの使用

設定ファイルから pod を作成することもできます。今回の [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/pods/pod.yaml) では上記で使用した `simpleservice` イメージと一般的な `CentOS` コンテナを一緒に実行します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pods/pod.yaml

$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
twocontainers             2/2       Running   0          7s
```

これで localhost から `CentOS` コンテナに exec を実行して、`simpleservice` にアクセスできます。

```bash
$ kubectl exec twocontainers -c shell -i -t -- bash
[root@twocontainers /]# curl -s localhost:9876/info
{"host": "localhost:9876", "version": "0.5.0", "from": "127.0.0.1"}
```

pod の中で `resources` フィールドを指定すると、CPU と RAM を [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/pods/constraint-pod.yaml) 内のコンテナがどの程度使用できるかを制限できます。(ここでは RAM が `64MB`、CPU が `0.5`)

```bash
$ kubectl create -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pods/constraint-pod.yaml

$ kubectl describe pod constraintpod
...
Containers:
  sise:
    ...
    Limits:
      cpu:      500m
      memory:   64Mi
    Requests:
      cpu:      500m
      memory:   64Mi
...
```

Kubernetes のリソース制限についてさらに学ぶには[ここ](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-ram-container/)と[ここ](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)のドキュメントをお読みください。

作成した pod をすべて削除するには以下を実行するだけです。

```bash
$ kubectl delete pod twocontainers

$ kubectl delete pod constraintpod
```

まとめると、Kubernetes で一つ以上のコンテナを (一緒に) 起動するのは簡単ですが、上記のように直接起動しようとすると致命的な限界があります。障害があってもコンテナの実行を維持するためには手動で管理しなければならないからです。それよりも良い方法は、[deployment](/deployments) で pod を管理することです。deployment を使えばライフサイクルをまたいで pod を制御できるため、新しいバージョンをロールアウトすることも可能です。

[次へ](/labels)
