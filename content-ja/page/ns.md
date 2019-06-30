+++
title = "Namespaces"
subtitle = "Kubernetes namespaces by example"
date = "2019-02-27"
url = "/ns/"
+++

名前空間はクラスタをより小さい単位に切り分けるために、 Kubernetes リソースのスコープを提供します。名前空間は他のユーザーと共有するワークスペースのように考えられます。pod や service のようなリソースの多くは名前空間で分けられますが、たとえば node のように名前空間で分けられずクラスタで共有されるリソースもあります。開発者は普段自分に割り当てられた名前空間を使いますが、管理者はアクセス制御やリソースクォータ設定などの用途で名前空間を管理したいと思うかもしれません。

全名前空間をリスト表示してみましょう。(出力は使用している環境によります。ここでは [Minishift](/diy/) を使っています)

```bash
$ kubectl get ns
NAME              STATUS    AGE
default           Active    13d
kube-system       Active    13d
namingthings      Active    12d
openshift         Active    13d
openshift-infra   Active    13d
```

名前空間についてさらに知るには `describe` コマンドを使います。以下が例です。

```bash
$ kubectl describe ns default
Name:   default
Labels: <none>
Status: Active

No resource quota.

No resource limits.
```

では、`test` という新しい[名前空間](https://github.com/openshift-evangelists/kbe/blob/master/specs/ns/ns.yaml)を作成してみましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/ns/ns.yaml
namespace "test" created

$ kubectl get ns
NAME              STATUS    AGE
default           Active    13d
kube-system       Active    13d
namingthings      Active    12d
openshift         Active    13d
openshift-infra   Active    13d
test              Active    3s
```

あるいは、`kubectl create namespace test` コマンドで名前空間を作ることもできます。

新しく作られた名前空間 `test` の中で [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/ns/pod.yaml) を起動するにはこうします。

```bash
$ kubectl apply --namespace=test -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/ns/pod.yaml
```

上記の方法を使うと、名前空間は実行時のプロパティになります。つまり、同じ pod や service などを複数の名前空間 (たとえば `dev` と `prod`) にデプロイできます。以下に示すように `metadata` セクションに名前空間を直接ハードコーディングすることもできますが、そうするとアプリのデプロイ時に柔軟性が欠けます。

```
apiVersion: v1
kind: Pod
metadata:
  name: podintest
  namespace: test
```

私たちの pod `podintest` のように名前空間で分けられたオブジェクトをリスト表示するには、次のコマンドを実行します。

```bash
$ kubectl get pods --namespace=test
NAME        READY     STATUS    RESTARTS   AGE
podintest   1/1       Running   0          16s
```

名前空間 (とその中のすべて) を削除するには、以下のようにします。

```bash
$ kubectl delete ns test
```

管理者であれば、名前空間を扱うためにより多くの情報を得るには [docs](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/) を読みたいと思われるかもしれません。

[前へ](/envs) | [次へ](/volumes)
