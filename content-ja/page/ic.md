+++
title = "Init Containers"
subtitle = "Kubernetes init containers by example"
date = "2019-02-26"
url = "/ic/"
+++

pod 内で実行するコンテナに準備が必要なことがあります。たとえば、サービスが利用可能になるのを待つ、実行時に何かを設定する、データベースのデータを初期化するといったことです。こういう場合に [init コンテナ](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) が役立ちます。留意点として、Kubernetes はすべての init コンテナを実行してから (それらが正常に終了してから)、メインのコンテナを実行します。

では [deployment](https://github.com/openshift-evangelists/kbe/blob/master/specs/ic/deploy.yaml) を作成しましょう。この中で init コンテナが `/ic/this` ファイルにメッセージを書き込み、メインの (実行状態を持続する) コンテナがこのファイルを読み込みます。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/ic/deploy.yaml
```

メインのコンテナの出力を確認しましょう。

```bash
$ kubectl get deploy,po
NAME                              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/ic-deploy   1         1         1            1           11m

NAME                            READY   STATUS    RESTARTS   AGE
pod/ic-deploy-bf75cbf87-8zmrb   1/1     Running   0          59s

$ kubectl logs ic-deploy-bf75cbf87-8zmrb -f
INIT_DONE
INIT_DONE
INIT_DONE
INIT_DONE
INIT_DONE
^C
```

init コンテナとその関連トピックについてさらに学ぶには、ブログ記事 [Kubernetes: A Pod’s Life](https://blog.openshift.com/kubernetes-pods-life/) を読んでください。


[前へ](/statefulset) | [次へ](/nodes)
