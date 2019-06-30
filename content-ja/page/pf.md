+++
title = "Port Forward"
subtitle = "Kubernetes port forward by example"
date = "2019-02-12"
url = "/pf/"
+++

Kubernetes 上でアプリを開発するときに、ローカル環境から service に手軽にアクセスできると便利です。わざわざロードバランサや ingress リソースを使ってサービスを公開するのは煩雑ですから。そういうときに使えるのが [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) です。

deployment と `simpleservice` という service から構成され、ポート `80` でサービス提供する [app](https://github.com/openshift-evangelists/kbe/blob/master/specs/pf/app.yaml) を作成しましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pf/app.yaml
```

ローカル環境、たとえばあなたのラップトップから、`simpleservice` service にポート番号 `8080` でアクセスしたいとしましょう。そのためにトラフィックを次のように転送します。

```bash
$ kubectl port-forward service/simpleservice 8080:80
Forwarding from 127.0.0.1:8080 -> 9876
Forwarding from [::1]:8080 -> 9876
```

上記から、トラフィックは service を介してポート番号 `9876` でサービス提供する pod にルーティングされていることがわかります。

これで、以下のように service をローカルから呼び出すことができます。(別のターミナルセッションで実行します)

```bash
$ curl localhost:8080/info
{"host": "localhost:8080", "version": "0.5.0", "from": "127.0.0.1"}
```

ポートフォワーディングは本番運用のトラフィックに使うべきではなく、開発用また実験用であることを覚えておいてくだあし。

[前へ](/sd) | [次へ](/healthz)
