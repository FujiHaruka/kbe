+++
title = "Logging"
subtitle = "Kubernetes logging by example"
date = "2019-02-28"
url = "/logging/"
+++

ロギングは、アプリケーションとクラスタ内部でおよそ何が起きているのかを理解するための一つの選択肢です。Kubernetes の基本的なロギングは、コンテナの出力を利用可能にします。これはデバッグにうってつけのユースケースです。より高度なものでは、[setups](http://some.ops4devs.info/logging/) は node 間を横断するログを扱い、それを中央に保存します。保存場所はクラスタ内部でもよいですが、専用の (クラウド) サービスを通じて保存することもできます。

`logme` という名前の [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/logging/pod.yaml) を作成しましょう。これは `stdout` と `stderr` に書き込むコンテナを実行します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/logging/pod.yaml
```

`logme` pod 内にある `gen` コンテナのログを最新 5 行表示するには、以下を実行します。

```bash
$ kubectl logs --tail=5 logme -c gen
Thu Apr 27 11:34:40 UTC 2017
Thu Apr 27 11:34:41 UTC 2017
Thu Apr 27 11:34:41 UTC 2017
Thu Apr 27 11:34:42 UTC 2017
Thu Apr 27 11:34:42 UTC 2017
```

`logme` pod 内にある `gen` コンテナのログを (`tail -f` のように) ストリーミングするには、以下を実行します。

```bash
$ kubectl logs -f --since=10s logme -c gen
Thu Apr 27 11:43:11 UTC 2017
Thu Apr 27 11:43:11 UTC 2017
Thu Apr 27 11:43:12 UTC 2017
Thu Apr 27 11:43:12 UTC 2017
Thu Apr 27 11:43:13 UTC 2017
...
```

なお、上のコマンドで `--since=10s` を指定しない場合、コンテナ起動後から書き込まれたログを全行取得できます。

ライフサイクルを終了した pod についてもログを確認できます。そのために `oneshot` という名前の [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/logging/oneshotpod.yaml) を作成します。これは 9 から 1 までカウントダウンしてから停止します。`-p` オプションを使うと、pod 内にあるコンテナの以前の (previous) インスタンスについてログを表示できます。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/logging/oneshotpod.yaml
$ kubectl logs -p oneshot -c gen
9
8
7
6
5
4
3
2
1
```

作成した pod を削除するには以下を実行します。

```bash
$ kubectl delete pod/logme pod/oneshot
```

[前へ](/secrets) | [次へ](/jobs)
