+++
title = "StatefulSet"
subtitle = "Kubernetes statefulset by example"
date = "2019-02-15"
url = "/statefulset/"
+++

ステートレスなアプリを運用するには deployment を使うのが適切です。けれども、ステートフルなアプリには [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) をお勧めします。deployment と違って、`StatefulSet` は管理下にある pod の同一性 (つまり予測可能な命名) と起動順序についてある保証を提供します。他に二点、deployment と違う点があります。一つは [headless services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) を作成するために必要なネットワーク通信で、もう一つは `StatefulSet` が管理する pod ごとの [persistent volume](/pv) の永続性です。

これがどのように互いに連動しているかを確認するために、[educational Kubernetes-native NoSQL datastore](https://blog.openshift.com/kubernetes-statefulset-in-action/) を使いましょう。

[stateful app](https://raw.githubusercontent.com/mhausenblas/mehdb/master/app.yaml) の作成から始めましょう。これは persistent volume と headless service をもっている `StatefulSet` です。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/mhausenblas/mehdb/master/app.yaml
```

1 分ほど待つと、すべてのリソースが作成済みになったのが確認できます。

```bash
$ kubectl get sts,po,pvc,svc
NAME                     DESIRED   CURRENT   AGE
statefulset.apps/mehdb   2         2         1m

NAME          READY   STATUS    RESTARTS   AGE
pod/mehdb-0   1/1     Running   0          1m
pod/mehdb-1   1/1     Running   0          56s

NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/data-mehdb-0   Bound    pvc-bc2d9b3b-310d-11e9-aeff-123713f594ec   1Gi        RWO            ebs            1m
persistentvolumeclaim/data-mehdb-1   Bound    pvc-d4b7620f-310d-11e9-aeff-123713f594ec   1Gi        RWO            ebs            56s

NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mehdb   ClusterIP   None         <none>        9876/TCP   1m
```

では、stateful app が適切に動作しているかどうか確認しましょう。そのために、headless service `mehdb:9876` の `/status` エンドポイントを使います。データストアにまだデータを入れていませんから、期待される結果は `0` 個のキーです。

```bash
$ kubectl run -it --rm jumpod --restart=Never --image=quay.io/mhausenblas/jump:0.2 -- curl mehdb:9876/status?level=full
If you don't see a command prompt, try pressing enter.
0
pod "jumpod" deleted
```

実際、上記のように `0` 個のキーが利用可能だとわかります。

なお、ステートフルなアプリに `StatefulSet` を使えば必ず万事解決となるとは限りません。ワークロードをきめ細かく制御するためにカスタムコントローラーを書いて、[custom resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) を定義したほうがうまくいく場合もあります。

[前へ](/jobs) | [次へ](/ic)
