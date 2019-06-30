+++
title = "Volumes"
subtitle = "Kubernetes volumes by example"
date = "2019-02-27"
url = "/volumes/"
+++

Kubernetes の volume は大雑把にいうと、pod 内で実行中のすべてのコンテナからアクセスできるディレクトリです。コンテナ内のファイルシステムとは対照的に、volume 内のデータはコンテナが再起動しても保持されます。volume の裏で使われる媒体と volume の内容は volume タイプによって決定されます。

- node 内で有効なタイプ (`emptyDir`、`hostPath` など)
- ファイル共有のタイプ (`nfs` など)
- クラウドプロバイダ特有のタイプ (`awsElasticBlockStore`、`azureDisk`、`gcePersistentDisk` など)
- 分散ファイルシステムのタイプ (たとえば `glusterfs`、`cephfs`)
- 特別な用途で使うタイプ (`secret`、`gitRepo` など)

特殊なタイプの volume として `PersistentVolume` がありますが、別のページで紹介します。

[pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/volumes/pod.yaml) を作成しましょう。2 つのコンテナが `emptyDir` volume を使ってデータを交換します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/volumes/pod.yaml

$ kubectl describe pod sharevol
Name:                   sharevol
Namespace:              default
...
Volumes:
  xchange:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
```

まず、pod 内のコンテナの一つ `c1` に exec を実行し、volume のマウントを確認し、データを生成しておきます。

```bash
$ kubectl exec -it sharevol -c c1 -- bash
[root@sharevol /]# mount | grep xchange
/dev/sda1 on /tmp/xchange type ext4 (rw,relatime,data=ordered)
[root@sharevol /]# echo 'some data' > /tmp/xchange/data
```

次に、pod 内で実行中の二つ目のコンテナ `c2` に exec を実行すると、`/tmp/data` に volume がマウントされ、前のステップで作成されたデータが読み込めることがわかります。

```bash
$ kubectl exec -it sharevol -c c2 -- bash
[root@sharevol /]# mount | grep /tmp/data
/dev/sda1 on /tmp/data type ext4 (rw,relatime,data=ordered)
[root@sharevol /]# cat /tmp/data/data
some data
```

留意点として、各コンテナのどこに volume をマウントするかを決める必要があり、また `emptyDir` では現在リソースの消費制限を設けることはできません。

pod を削除するには以下を実行します。

```bash
$ kubectl delete pod/sharevol
```

前述のように、共有 volume とその内容はすべて削除されます。

[前へ](/ns) | [次へ](/pv)
