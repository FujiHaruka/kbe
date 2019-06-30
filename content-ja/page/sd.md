+++
title = "Service Discovery"
subtitle = "Kubernetes service discovery by example"
date = "2019-02-27"
url = "/sd/"
+++

service discovery は [service](/service/) に接続する方法を見つけるプロセスです。[環境変数](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/#environment-variables)ベースの servcice discovery オプションがありますが、DNS ベースの service discovery のほうがお勧めです。DNS は[クラスタアドオン](https://github.com/kubernetes/kubernetes/blob/master/cluster/addons/dns/README.md)なので、Kubernetes ディストリビューションがそのアドオンを提供していることを確認してください。なければご自身でインストールしてください。

`thesvc` という名前の [service](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/svc.yaml) と、それに伴っていくつかの pod を管理する [RC](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/rc.yaml) を作成しましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/rc.yaml

$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/svc.yaml
```

クラスタ内部から、たとえば別の service から `thesvc` service に接続したいとします。これをシミュレートするために、同じ名前空間内に [jump pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/jumpod.yaml) を作ります。(名前空間は特に指定していないので `default` です)

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/jumpod.yaml
```

DNS アドオンは `thesvc` service が FQDN `thesvc.default.svc.cluster.local` を通じてクラスタ内の他の pod からアクセス可能であることを保証してくれます。試してみましょう。

```bash
$ kubectl exec -it jumpod -c shell -- ping thesvc.default.svc.cluster.local
PING thesvc.default.svc.cluster.local (172.30.251.137) 56(84) bytes of data.
...
```

`ping` の結果から、service がクラスタ IP `172.30.251.137` によってアクセス可能であることがわかります。次のように直接接続して service を利用することも (同じ名前空間内であれば) 可能です。

 ```bash
 $ kubectl exec -it jumpod -c shell -- curl http://thesvc/info
{"host": "thesvc", "version": "0.5.0", "from": "172.17.0.5"}
```

上の IP アドレス `172.17.0.5` は、 jump pod のクラスタ内での IP アドレスです。

別の名前空間にデプロイされた service にアクセスするには、`$SVC.$NAMESPACE.svc.cluster.local` という形式の FQDN を使用します。

どういう仕組みか理解するために、以下を作成しましょう。

1. [名前空間](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/other-ns.yaml) `other`
1. 名前空間 `other` 内の [service](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/other-svc.yaml) `thesvc`
1. 名前空間 `other` 内で pod を管理する [RC](https://github.com/openshift-evangelists/kbe/blob/master/specs/sd/other-rc.yaml)

名前空間に慣れていなければ、[namespace examples](/ns/) を先に読んでみてください。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/other-ns.yaml

$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/other-rc.yaml

$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/sd/other-svc.yaml
```

これで、名前空間 `other` 内の `thesvc` service に名前空間 `default` から (再度 jump pod を通じて) アクセスできるようになりました。

 ```bash
$ kubectl exec -it jumpod -c shell -- curl http://thesvc.other/info
{"host": "thesvc.other", "version": "0.5.0", "from": "172.17.0.5"}
```

まとめると、DNS ベースの service discovery は、クラスタ内を横断して service に接続する柔軟で一般的な方法を提供します。

作成したリソースをすべて削除するには以下を実行します。

```bash
$ kubectl delete pods jumpod

$ kubectl delete svc thesvc

$ kubectl delete rc rcsise

$ kubectl delete ns other
```

名前空間を削除するとその中のすべてのリソースが削除されることを覚えておいてください。

[前へ](/services) | [次へ](/pf)
