+++
title = "Deployments"
subtitle = "Kubernetes deployments by example"
date = "2019-02-27"
url = "/deployments/"
+++

deployment は [pod](/pods/) のスーパーバイザです。deployment を使うと、新バージョンの pod をいつどのようにロールアウトし、また以前の状態にロールバックするかをきめ細かく制御できます。

[deployment](https://github.com/openshift-evangelists/kbe/blob/master/specs/deployments/d09.yaml) を作りましょう。名前は `sise-deploy` で、2 つの pod の replica と 1 つの replica set を管理します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/deployments/d09.yaml
```

deployment、replica set、そして deployment が面倒を見ている pod を次のように確認できます。

```bash
$ kubectl get deploy
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
sise-deploy   2         2         2            2           10s

$ kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
sise-deploy-3513442901   2         2         2         19s

$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
sise-deploy-3513442901-cndsx   1/1       Running   0          25s
sise-deploy-3513442901-sn74v   1/1       Running   0          25s
```

pod と replica set の命名は deployment の名前を元にしています。

この時点で、pod の中で実行されている `sise` コンテナはバージョン `0.9` を返すように設定されています。クラスタの内部から確認してみましょう。(pod の IP を取得するには `kubectl describe` を使います)

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.9", "from": "172.17.0.1"}
```

ではバージョンを `1.0` に変えるとどうなるか確認しましょう。[deployment](https://github.com/openshift-evangelists/kbe/blob/master/specs/deployments/d10.yaml) を更新します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/deployments/d10.yaml
deployment "sise-deploy" configured
```

手動で deployment を編集する代わりに `kubectl edit deploy/sise-deploy` を使えることに留意してください。

これでわかりますが、バージョン `1.0` に更新された新しい pod が 2 つロールアウトされ、バージョン `0.9` の古い pod が 2 つ停止しました。

```bash
$ kubectl get pods
NAME                           READY     STATUS        RESTARTS   AGE
sise-deploy-2958877261-nfv28   1/1       Running       0          25s
sise-deploy-2958877261-w024b   1/1       Running       0          25s
sise-deploy-3513442901-cndsx   1/1       Terminating   0          16m
sise-deploy-3513442901-sn74v   1/1       Terminating   0          16m
```

また、新しい replica set が deployment によって作成されました。

```bash
$ kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
sise-deploy-2958877261   2         2         2         4s
sise-deploy-3513442901   0         0         0         24m
```

deployment 中に進行を確認するには `kubectl rollout status deploy/sise-deploy` が使えます。

新しい `1.0` バージョンが本当に利用可能かどうかを確認するために、クラスタの内部から以下を実行します。(再掲。pod の IP を取得するには `kubectl describe` を使います)

```bash
[cluster] $ curl 172.17.0.5:9876/info
{"host": "172.17.0.5:9876", "version": "1.0", "from": "172.17.0.1"}
```

すべての deployment の履歴は次のように取得できます。

```bash
$ kubectl rollout history deploy/sise-deploy
deployments "sise-deploy"
REVISION        CHANGE-CAUSE
1               <none>
2               <none>
```

deployment に問題があれば、Kubernetes は自動的に前のバージョンにロールバックしますが、特定のリビジョンに明示的にロールバックすることもできます。この例ではリビジョン 1 (元の pod バージョン)にロールバックします。

```bash
$ kubectl rollout undo deploy/sise-deploy --to-revision=1
deployment "sise-deploy" rolled back

$ kubectl rollout history deploy/sise-deploy
deployments "sise-deploy"
REVISION        CHANGE-CAUSE
2               <none>
3               <none>

$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
sise-deploy-3513442901-ng8fz   1/1       Running   0          1m
sise-deploy-3513442901-s8q4s   1/1       Running   0          1m
```

この時点で最初の状態に戻りました。2 つの新しい pod がまたバージョン `0.9` で実行されています。

最後に、クリーンアップするため deployment を削除します。deployment が管理している replica set と pod も一緒に削除されます。

```bash
$ kubectl delete deploy sise-deploy
deployment "sise-deploy" deleted
```

deployment の他のオプションと、それがいつトリガーされるかについては、[ドキュメント](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)も参考にしてください。

[前へ](/labels) | [次へ](/services)
