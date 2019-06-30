+++
title = "Persistent Volumes"
subtitle = "Kubernetes persistent volumes by example"
date = "2019-02-27"
url = "/pv/"
+++

[persistent volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) はクラスタで共通のリソースで、pod が削除されてもデータを維持したいときに使えます。PV の裏の仕組みは worker node 上のローカル接続ストレージではなく、EBS や NFS のようなネットワークストレージシステムや、あるいは Ceph のような分散ファイルシステムです。

PV を使うためには、最初に persistent volume claim (PVC) で宣言する必要があります。PVC は PV に望ましい仕様 (サイズ、速度など) を Kubernetes からリクエストし、それからそれを volume としてマウントできる pod にバインドします。では、[PVC](https://github.com/openshift-evangelists/kbe/blob/master/specs/pv/pvc.yaml) を作成しましょう。Kubernetes に 1 GB のストレージをデフォルトの [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) で要求します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pv/pvc.yaml

$ kubectl get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
myclaim   Bound    pvc-27fed6b6-3047-11e9-84bb-12b5519f9b58   1Gi        RWO            gp2-encrypted   18m
```

永続性の仕組みを理解するために、上の PVC を使用してそれを `/tmp/persistent` に volume としてマウントする [deployment](https://github.com/openshift-evangelists/kbe/blob/master/specs/pv/deploy.yaml) を作成しましょう。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/pv/deploy.yaml
```

さて、volume が実際に永続化されているかテストしたいと思います。そのために、上の deployment で管理している pod を見つけ、そのメインのコンテナに exec を実行して、`/tmp/persistent` ディレクトリ (PV をマウントすることにしたディレクトリ) に `data` という名前のファイルを作成します。

```bash
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
pv-deploy-69959dccb5-jhxx    1/1     Running   0          16m

$ kubectl exec -it pv-deploy-69959dccb5-jhxxw -- bash
bash-4.2$ touch /tmp/persistent/data
bash-4.2$ ls /tmp/persistent/
data  lost+found
```

今度は、pod を削除して deployment に新しい pod を起動させます。期待されることは、PV が新しい pod でも利用可能で、`/tmp/persistent` 内のデータがまだ存在していることです。確認しましょう。

```bash
$ kubectl delete po pv-deploy-69959dccb5-jhxxw
pod pv-deploy-69959dccb5-jhxxw deleted

$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
pv-deploy-69959dccb5-kwrrv   1/1     Running   0          16m

$ kubectl exec -it pv-deploy-69959dccb5-kwrrv -- bash
bash-4.2$ ls /tmp/persistent/
data  lost+found
```

実際、`data` ファイルとその中身が期待される場所にきちんとあります。

留意点として、デフォルトの動作では deployment が削除されても PVC (と PV) は存在し続けます。ストレージをこのように保護する機能はデータ損失を防ぐためです。データがもう確実に不要になったら、先に進んで PVC を削除することができます。それに伴って PV も削除されます。

```bash
$ kubectl delete pvc myclaim
persistentvolumeclaim "myclaim" deleted
```

どんなタイプの PV が Kubernetes クラスタで利用可能かは環境によります (オンプレミスかパブリッククラウドか)。このトピックについてさらに学びたい場合は、[Stateful Kubernetes](https://stateful.kubernetes.sh/#storage) を読んでください。

[前へ](/volumes) | [次へ](/secrets)
