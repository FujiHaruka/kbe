+++
title = "Secrets"
subtitle = "Kubernetes secrets by example"
date = "2019-02-28"
url = "/secrets/"
+++

データベースのパスワードや API キーなどの機密情報は見える形で保存したくないものです。そういった機密情報を安全で信頼できる方法で扱うための機構を提供するのが secret です。secret には次のような性質があります。

- secret は名前空間ごとのオブジェクトです。つまり、名前空間のコンテキストの中に存在します
- pod 内のコンテナから secret にアクセスするには volume や環境変数を通じて行えます
- node 上の secret のデータは [tmpfs](https://www.kernel.org/doc/Documentation/filesystems/tmpfs.txt) volume に保存されます
- 1 つの secret のサイズは上限 1 MB です
- API サーバーは secret を etcd 内に平文で保存します

`apiKey` という名前の secret を作成しましょう。これは (発行済みの) API キーを保持します。

```bash
$ echo -n "A19fh68B001j" > ./apikey.txt

$ kubectl create secret generic apikey --from-file=./apikey.txt
secret "apikey" created

$ kubectl describe secrets/apikey
Name:           apikey
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:   Opaque

Data
====
apikey.txt:     12 bytes
```

では、[pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/secrets/pod.yaml) 内の secret を [volume](/volumes/) 経由で使ってみましょう。


```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/secrets/pod.yaml
```

コンテナに exec を実行すると、secret が `/tmp/apikey` にマウントされているのがわかります。

```
$ kubectl exec -it consumesec -c shell -- bash
[root@consumesec /]# mount | grep apikey
tmpfs on /tmp/apikey type tmpfs (ro,relatime)
[root@consumesec /]# cat /tmp/apikey/apikey.txt
A19fh68B001j
```

サービスアカウント用には、Kubernetes は API にアクセスするための認証情報を含む secret を自動的に作成し、このタイプの secret を使うために pod を修正してくれます。

pod と secret を削除するには以下を実行します。

```bash
$ kubectl delete pod/consumesec secret/apikey
```

[前へ](/volumes) | [次へ](/logging)
