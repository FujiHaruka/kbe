+++
title = "API Server access"
subtitle = "Kubernetes API Server access by example"
date = "2019-02-13"
url = "/api/"
+++

調査やテストのために [Kubernetes API サーバー](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/)に直接アクセスできると便利な場合や、それが必要な場合があります。

そうするための一つの選択肢が、ローカル環境に API をプロキシすることです。以下のようにします。

```bash
$ kubectl proxy --port=8080
Starting to serve on 127.0.0.1:8080

```

API にクエリを送るには (別のターミナルセッションで) 以下のようにします。

```bash
$ curl http://localhost:8080/api/v1
{
  "kind": "APIResourceList",
  "groupVersion": "v1",
  "resources": [
    {
...
    {
      "name": "services/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Service",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}
```

あるいは、プロキシせずに以下のように直接 `kubectl` を使っても同じことができます。

```bash
$ kubectl get --raw=/api/v1
```

また、サポートしている API のバージョンやリソースを調べたければ、以下のコマンドが使えます。

```bash
$ kubectl api-versions
admissionregistration.k8s.io/v1beta1
...
v1

$ kubectl api-resources
NAME                                  SHORTNAMES     APIGROUP                       NAMESPACED   KIND
bindings                                                                            true         Binding
...
storageclasses                        sc             storage.k8s.io                 false        StorageClass
```

[前へ](/nodes)
