+++
title = "DIY"
subtitle = "Try out for yourself"
date = "2019-02-28"
url = "/diy/"
+++

ご自分でサンプルを実行してみたい場合、私がローカルマシン上で実行したのと同じセットアップが利用できます。

1. [Minishift](https://docs.okd.io/latest/minishift/getting-started/installing.html) をインストールします
1. `minishift start` を実行します
1. OpenShift Client バイナリ (`oc`) を `eval $(minishift oc-env)` を実行することで設定します
1. `oc login -u system:admin` でログインします。パスワードは `admin` です
1. `oc` から次のようにシンボリックリンクを作成します。`ln -s $(which oc) /usr/local/bin/kubectl`

すべてのサンプルは Minishift のバージョン [v1.32.0](https://github.com/minishift/minishift/releases/tag/v1.32.0) で実行されました。

```bash
$ minishift version
minishift v1.32.0+009893b

$ kubectl version --short
Client Version: v1.12.2
Server Version: v1.11.0+d4cacc0
```

あるいは、ここのサンプルを使って [OpenShift 4](https://try.openshift.com/) の開発者プレビューを試すこともできます。
