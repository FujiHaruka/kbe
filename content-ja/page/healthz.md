+++
title = "Health Checks"
subtitle = "Kubernetes health checks by example"
date = "2019-02-27"
url = "/healthz/"
+++

pod 内のコンテナが正常でトラフィックの処理を受け付ける状態にあるかどうかを確認するために、Kubernetes はさまざまなヘルスチェック機構を提供しています。ヘルスチェックは Kubernetes で **probe** と呼ばれます。[kubelet](https://kubernetes.io/docs/admin/kubelet/) が実行する `livenessProbe` はいつコンテナを再起動するかを決定し、service と deployment が使用する `readinessProbe` は pod がトラフィックを受け取るべきかどうかを決定します。

ここでは次のように HTTP ヘルスチェックをよく見ることにしましょう。留意点として、コンテナが正常かどうか (そして潜在的に利用できるかどうか) を判断するために kubelet が使用する URL を公開しておくのは、アプリケーション開発者の責任です。

[pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/healthz/pod.yaml) を作成しましょう。これはエンドポイント `/health` を公開していて、HTTP ステータスコード `200` を返します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/healthz/pod.yaml
```

pod の仕様定義の中で、次のように定義しました。

```
livenessProbe:
  initialDelaySeconds: 2
  periodSeconds: 5
  httpGet:
    path: /health
    port: 9876
```

上記の意味を説明すると、Kubernetes は `/health` エンドポイントのチェックを最初に 2 秒待ってから 5 秒ごとに開始するということです。

pod を見てみると、正常状態と診断されていることがわかります。

```bash
$ kubectl describe pod hc
Name:                   hc
Namespace:              default
Security Policy:        anyuid
Node:                   192.168.99.100/192.168.99.100
Start Time:             Tue, 25 Apr 2017 16:21:11 +0100
Labels:                 <none>
Status:                 Running
...
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath           Type            Reason          Message
  ---------     --------        -----   ----                            -------------           --------        ------          -------
  3s            3s              1       {default-scheduler }                                    Normal          Scheduled       Successfully assigned hc to 192.168.99.100
  3s            3s              1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Pulled          Container image "mhausenblas/simpleservice:0.5.0"
already present on machine
  3s            3s              1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Created         Created container with docker id 8a628578d6ad; Security:[seccomp=unconfined]
  2s            2s              1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Started         Started container with docker id 8a628578d6ad
```

では、[bad pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/healthz/badpod.yaml) を起動します。この pod は、ランダムに (1秒間から4秒間) ステータスコード 200 を返さないコンテナをもっています。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/healthz/badpod.yaml
```

bad pod のイベントを見てみると、ヘルスチェックが失敗したことがわかります。

```bash
$ kubectl describe pod badpod
...
Events:
  FirstSeen     LastSeen        Count   From                            SubobjectPath           Type            Reason          Message
  ---------     --------        -----   ----                            -------------           --------        ------          -------
  1m            1m              1       {default-scheduler }                                    Normal          Scheduled       Successfully assigned badpod to 192.168.99.100
  1m            1m              1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Created         Created container with docker id 7dd660f04945; Security:[seccomp=unconfined]
  1m            1m              1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Started         Started container with docker id 7dd660f04945
  1m            23s             2       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Pulled          Container image "mhausenblas/simpleservice:0.5.0" already present on machine
  23s           23s             1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Killing         Killing container with docker id 7dd660f04945: pod "badpod_default(53e5c06a-29cb-11e7-b44f-be3e8f4350ff)" container "sise" is unhealthy, it will be killed and re-created.
  23s           23s             1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Created         Created container with docker id ec63dc3edfaa; Security:[seccomp=unconfined]
  23s           23s             1       {kubelet 192.168.99.100}        spec.containers{sise}   Normal          Started         Started container with docker id ec63dc3edfaa
  1m            18s             4       {kubelet 192.168.99.100}        spec.containers{sise}   Warning         Unhealthy       Liveness probe failed: Get http://172.17.0.4:9876/health: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
```

これは以下のようにも確認できます。

```bash
$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
badpod                    1/1       Running   4          2m
hc                        1/1       Running   0          6m
```

上記から、`badpod` はすでに 4 回再起動したことがわかります。ヘルスチェックが失敗したからです。

`livenessProbe` に加えて `readinessProbe` も定義できます。これは同じやり方で設定できますが、ユースケースとセマンティクスは異なります。`readinessProbe` は pod 内のコンテナの起動段階をチェックするのに使います。S3 のような外部ストレージからデータをロードするコンテナや、テーブルの初期化が必要なデータベースを想像してみてください。こういうときに、コンテナがトラフィックを処理する準備ができたら知らせてほしくなります。

`readinessProbe` が 10 秒後に開始する [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/healthz/ready.yaml) を作成しましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/healthz/ready.yaml
```

pod のイベントを見ると、しばらくあとで pod がトラフィックを処理する準備ができたことがわかります。

```bash
$ kubectl describe pod ready
...
Conditions:                                                                                                                                                               [0/1888]
  Type          Status
  Initialized   True
  Ready         True
  PodScheduled  True
...
```
作成した pod をすべて削除するには以下のようにします。

```bash
$ kubectl delete pod/hc pod/ready pod/badpod
```

TCP や command probe など、probe の設定についてさらに学ぶには [docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) を参照してください。

[前へ](/sd) | [次へ](/envs)
