+++
title = "Labels"
subtitle = "Kubernetes labels by example"
date = "2019-02-27"
url = "/labels/"
+++

label は Kubernetes オブジェクトを組織化するための仕組みです。label は key/value のペアで、長さと許可される値にはある[制約](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)が課せられますが、事前に意味は定義されていません。そのため、label は目的に合わせて自由に選べます。たとえば、「この pod はプロダクション環境で実行されている」といった環境を表現したり、「あの pod は X 課が所有している」といった所有権を表現したりできます。

初期状態で一つの label (`env=development`) をもつ [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/labels/pod.yaml) を作成しましょう。


```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/labels/pod.yaml

$ kubectl get pods --show-labels
NAME       READY     STATUS    RESTARTS   AGE    LABELS
labelex    1/1       Running   0          10m    env=development
```
上の `get pods` に `--show-labels` オプションを付けました。このオプションを指定すると、出力にオブジェクトの label 用の列を追加します。

次のように label をこの pod に追加できます。

```bash
$ kubectl label pods labelex owner=michael

$ kubectl get pods --show-labels
NAME        READY     STATUS    RESTARTS   AGE    LABELS
labelex     1/1       Running   0          16m    env=development,owner=michael
```

label でフィルターするには、`--selector` オプションを使用します。たとえば、`owner` が `michael` と等しい pod だけをリスト表示するには、次のようにします。

```bash
$ kubectl get pods --selector owner=michael
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

`--selector` オプションは `-l` に短縮できます。`env=development` という label の pod だけを選ぶにはこうします。

```bash
$ kubectl get pods -l env=development
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

Kubernetes オブジェクトは複数の値を扱う set-based selector もサポートしています。もう一つの [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/labels/anotherpod.yaml) を起動しましょう。二つの label が (`env=production` と `owner=michael`) 付けられています。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/labels/anotherpod.yaml
```

では、`env=development` または `env=production` どちらかの label が付いた pod をリスト表示しましょう。

```bash
$ kubectl get pods -l 'env in (production, development)'
NAME           READY     STATUS    RESTARTS   AGE
labelex        1/1       Running   0          43m
labelexother   1/1       Running   0          3m
```

label selection は他のコマンドでもサポートされています。たとえば、これまでの pod を削除するには次のようにします。

```bash
$ kubectl delete pods -l 'env in (production, development)'
```

指定した label をもつ pod がすべて削除されるので留意してください。

これらの pod を名前を指定して直接削除することもできます。

```bash
$ kubectl delete pods labelex

$ kubectl delete pods labelexother
```

label を付けられるのは pod だけではありません。node や service といったどんなオブジェクトにも label を適用できます。

[前へ](/pods) | [次へ](/deployments)
