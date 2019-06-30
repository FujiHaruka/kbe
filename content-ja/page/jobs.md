+++
title = "Jobs"
subtitle = "Kubernetes jobs by example"
date = "2019-02-28"
url = "/jobs/"
+++

Kubernetes の [job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) はバッチ処理を実行する pod を管理します。ある時間になると実行されるプロセスで、たとえば計算やバックアップ操作を完了するために実行されます。

`countdown` という [job](https://github.com/openshift-evangelists/kbe/blob/master/specs/jobs/job.yaml) を作成しまそふ。これは 9 から 1 にカウントダウンする pod を管理します。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/jobs/job.yaml
```

job とそれが面倒を見ている pod は次のように確認できます。

```bash
$ kubectl get jobs
NAME                      DESIRED   SUCCESSFUL   AGE
countdown                 1         1            5s

$ kubectl get pods --show-all
NAME                            READY     STATUS      RESTARTS   AGE
countdown-lc80g                 0/1       Completed   0          16s
```

job のステータスについてさらに知るには、こうします。

```bash
$ kubectl describe jobs/countdown
Name:           countdown
Namespace:      default
Image(s):       centos:7
Selector:       controller-uid=ff585b92-2b43-11e7-b44f-be3e8f4350ff
Parallelism:    1
Completions:    1
Start Time:     Thu, 27 Apr 2017 13:21:10 +0100
Labels:         controller-uid=ff585b92-2b43-11e7-b44f-be3e8f4350ff
                job-name=countdown
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
No volumes.
Events:
  FirstSeen     LastSeen        Count   From                    SubobjectPath   Type            Reason                  Message
  ---------     --------        -----   ----                    -------------   --------        ------                  -------
  2m            2m              1       {job-controller }                       Normal          SuccessfulCreate        Created pod: countdown-lc80g
```

そして、job の出力をそれが管理している pod を通じて確認するには、以下を実行します。

```bash
kubectl logs countdown-lc80g
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

クリーンアップするには、`delete` コマンドを job オブジェクトに使います。job が管理している pod もすべて削除されます。

```bash
$ kubectl delete job countdown
job "countdown" deleted
```

job の使用には他にも高度な方法があります。たとえば、[work queue](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/) を活用したり、[cron jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) で特定の時間に実行するようスケジュールしたりできます。

[前へ](/logging) | [次へ](/statefulset)
