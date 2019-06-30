+++
title = "Services"
subtitle = "Kubernetes services by example"
date = "2019-02-27"
url = "/services/"
+++

service は pod の抽象化で、いわゆる仮想 IP (VIP) アドレスを安定して提供します。pod は変更があるとそれに伴う IP アドレスも変更される可能性がありますが、service は VIP を使うため pod の中で実行するコンテナにクライアントから確実に接続できます。VIP の `virtual` とはどういうことかというと、ネットワークインターフェイスに結びついた実際の IP アドレスではありません。その目的は、単純に一つ以上の pod にトラフィックを転送することです。VIP と pod のマッピングを最新状態で維持するのは、[kube-proxy](https://kubernetes.io/docs/admin/kube-proxy/) の仕事です。kube-proxy は各 node 上で実行されるプロセスで、クラスタ内の新しい service について知るために API サーバーに問い合わせます。

[RC](https://github.com/openshift-evangelists/kbe/blob/master/specs/services/rc.yaml) と [service](https://github.com/openshift-evangelists/kbe/blob/master/specs/services/svc.yaml) によって管理された pod を作成してみましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/services/rc.yaml

$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/services/svc.yaml
```

管理されている pod が実行されているのを確認できます。

```bash
$ kubectl get pods -l app=sise
NAME           READY     STATUS    RESTARTS   AGE
rcsise-6nq3k   1/1       Running   0          57s

$ kubectl describe pod rcsise-6nq3k
Name:                   rcsise-6nq3k
Namespace:              default
Security Policy:        restricted
Node:                   localhost/192.168.99.100
Start Time:             Tue, 25 Apr 2017 14:47:45 +0100
Labels:                 app=sise
Status:                 Running
IP:                     172.17.0.3
Controllers:            ReplicationController/rcsise
Containers:
...
```

pod に割り当てられた IP `172.17.0.3` を通じてクラスタの内部から pod に直接アクセスできます。

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.5.0", "from": "172.17.0.1"}
```

ただし、これは前述のように pod に割り当てられた IP アドレスが変わる可能性があるためお勧めできません。そこで、作成した `simpleservice` に入ります。

```bash
$ kubectl get svc
NAME              CLUSTER-IP       EXTERNAL-IP   PORT(S)                   AGE
simpleservice     172.30.228.255   <none>        80/TCP                    5m

$ kubectl describe svc simpleservice
Name:                   simpleservice
Namespace:              default
Labels:                 <none>
Selector:               app=sise
Type:                   ClusterIP
IP:                     172.30.228.255
Port:                   <unset> 80/TCP
Endpoints:              172.17.0.3:9876
Session Affinity:       None
No events.
```

service は label を通じて、トラフィックの転送先となる pod を追跡します。私たちの例では、label は `app=sise` です。

クラスタの内部から `simpleservice` にアクセスできます。

```bash
[cluster] $ curl 172.30.228.255:80/info
{"host": "172.30.228.255", "version": "0.5.0", "from": "10.0.2.15"}
```

どうやって VIP `172.30.228.255` のトラフィックを pod に転送させているのでしょうか。答えは [IPtables](https://wiki.centos.org/HowTos/Network/IPTables) です。IPtables は簡単にいうと Linux カーネルに特定の IP パッケージをどう処理するかを教える長大なリストです。

私たちのサービスに関するルールを見ます。(クラスタノード上で実行します)

```bash
[cluster] $ sudo iptables-save | grep simpleservice
-A KUBE-SEP-4SQFZS32ZVMTQEZV -s 172.17.0.3/32 -m comment --comment "default/simpleservice:" -j KUBE-MARK-MASQ
-A KUBE-SEP-4SQFZS32ZVMTQEZV -p tcp -m comment --comment "default/simpleservice:" -m tcp -j DNAT --to-destination 172.17.0.3:9876
-A KUBE-SERVICES -d 172.30.228.255/32 -p tcp -m comment --comment "default/simpleservice: cluster IP" -m tcp --dport 80 -j KUBE-SVC-EZC6WLOVQADP4IAW
-A KUBE-SVC-EZC6WLOVQADP4IAW -m comment --comment "default/simpleservice:" -j KUBE-SEP-4SQFZS32ZVMTQEZV
```

上を見ると、ありがたいことに `kube-proxy` がルーティングテーブルに 4 つのルールを追加したことがわかります。そのルールが述べていることをかいつまむと、`172.30.228.255:80` に来る TCP トラフィックを `172.17.0.3:9876` に、つまり私たちの pod に転送しなさいということです。

次に、pod を管理している RC をスケールアップして、二番目の pod を追加しましょう。

```bash
$ kubectl scale --replicas=2 rc/rcsise
replicationcontroller "rcsise" scaled

$ kubectl get pods -l app=sise
NAME           READY     STATUS    RESTARTS   AGE
rcsise-6nq3k   1/1       Running   0          15m
rcsise-nv8zm   1/1       Running   0          5s
```

ルーティングテーブルの関連する部分をもう一度確認すると、IPtables のルールが追加されているのがわかります。

```bash
[cluster] $ sudo iptables-save | grep simpleservice
-A KUBE-SEP-4SQFZS32ZVMTQEZV -s 172.17.0.3/32 -m comment --comment "default/simpleservice:" -j KUBE-MARK-MASQ
-A KUBE-SEP-4SQFZS32ZVMTQEZV -p tcp -m comment --comment "default/simpleservice:" -m tcp -j DNAT --to-destination 172.17.0.3:9876
-A KUBE-SEP-PXYYII6AHMUWKLYX -s 172.17.0.4/32 -m comment --comment "default/simpleservice:" -j KUBE-MARK-MASQ
-A KUBE-SEP-PXYYII6AHMUWKLYX -p tcp -m comment --comment "default/simpleservice:" -m tcp -j DNAT --to-destination 172.17.0.4:9876
-A KUBE-SERVICES -d 172.30.228.255/32 -p tcp -m comment --comment "default/simpleservice: cluster IP" -m tcp --dport 80 -j KUBE-SVC-EZC6WLOVQADP4IAW
-A KUBE-SVC-EZC6WLOVQADP4IAW -m comment --comment "default/simpleservice:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-4SQFZS32ZVMTQEZV
-A KUBE-SVC-EZC6WLOVQADP4IAW -m comment --comment "default/simpleservice:" -j KUBE-SEP-PXYYII6AHMUWKLYX
```

上のルーティングテーブルのリストをみると、新しく作られた pod は `172.17.0.4:9876` をサーブしていて、その pod 用のルールが追加されたのがわかりますが、他にも一つルーツが追加されています。

```
-A KUBE-SVC-EZC6WLOVQADP4IAW -m comment --comment "default/simpleservice:" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-4SQFZS32ZVMTQEZV
```

このルールによって、IPtables の `statistics` モジュールが呼び出され、service に来るトラフィックが 2 つの pod に平等に分配されます。

すべてのリソースを削除するには以下を実行します。

```bash
$ kubectl delete svc simpleservice

$ kubectl delete rc rcsise
```

[前へ](/deployments) | [次へ](/sd)
