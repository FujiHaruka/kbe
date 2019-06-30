+++
title = "Nodes"
subtitle = "Kubernetes nodes by example"
date = "2019-02-28"
url = "/nodes/"
+++

Kubernetes において、node は pod の形をしたワークロードを実行する (仮想) マシンです。普通、開発者は node を直接扱いませんが、管理者は node の[操作](https://kubernetes.io/docs/concepts/nodes/node/)に慣れておくとよいでしょう。

クラスタ内で利用可能な node をリスト表示するには、以下を実行します。(出力は環境によって異なります。ここでは [Minishift](/diy/) を使用しています)

```bash
$ kubectl get nodes
NAME             STATUS    AGE
192.168.99.100   Ready     14d
```

開発者の視点から興味深いタスクは、Kubernetes に特定の node 上の pod をスケジュールさせることです。そのためにまず、対象の node に label を付ける必要があります。

```bash
$ kubectl label nodes 192.168.99.100 shouldrun=here
node "192.168.99.100" labeled
```

これで [pod](https://github.com/openshift-evangelists/kbe/blob/master/specs/nodes/pod.yaml) を作成できます。この pod は `shouldrun=here` という label の付いた node 上にスケジュールされます。

```bash
$ kubectl apply -f https://raw.githubusercontent.com/openshift-evangelists/kbe/master/specs/nodes/pod.yaml

$ kubectl get pods --output=wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
onspecificnode            1/1       Running   0          8s        172.17.0.3       192.168.99.100
```

特定の node について、たとえば `192.168.99.100` について詳細を知るには以下を実行します。

```bash
$ kubectl describe node 192.168.99.100
Name:			192.168.99.100
Labels:			beta.kubernetes.io/arch=amd64
			beta.kubernetes.io/os=linux
			kubernetes.io/hostname=192.168.99.100
			shouldrun=here
Taints:			<none>
CreationTimestamp:	Wed, 12 Apr 2017 17:17:13 +0100
Phase:
Conditions:
  Type			Status	LastHeartbeatTime			LastTransitionTime			Reason				Message
  ----			------	-----------------			------------------			------				-------
  OutOfDisk 		False 	Thu, 27 Apr 2017 14:55:49 +0100 	Thu, 27 Apr 2017 09:18:13 +0100 	KubeletHasSufficientDisk 	kubelet has sufficient disk space available
  MemoryPressure 	False 	Thu, 27 Apr 2017 14:55:49 +0100 	Wed, 12 Apr 2017 17:17:13 +0100 	KubeletHasSufficientMemory 	kubelet has sufficient memory available
  DiskPressure 		False 	Thu, 27 Apr 2017 14:55:49 +0100 	Wed, 12 Apr 2017 17:17:13 +0100 	KubeletHasNoDiskPressure 	kubelet has no disk pressure
  Ready 		True 	Thu, 27 Apr 2017 14:55:49 +0100 	Thu, 27 Apr 2017 09:18:24 +0100 	KubeletReady 			kubelet is posting ready status
Addresses:		192.168.99.100,192.168.99.100,192.168.99.100
Capacity:
 alpha.kubernetes.io/nvidia-gpu:	0
 cpu:					2
 memory:				2050168Ki
 pods:					20
Allocatable:
 alpha.kubernetes.io/nvidia-gpu:	0
 cpu:					2
 memory:				2050168Ki
 pods:					20
System Info:
 Machine ID:			896b6d970cd14d158be1fd1c31ff1a8a
 System UUID:			F7771C31-30B0-44EC-8364-B3517DBC8767
 Boot ID:			1d589b36-3413-4e82-af80-b2756342eed4
 Kernel Version:		4.4.27-boot2docker
 OS Image:			CentOS Linux 7 (Core)
 Operating System:		linux
 Architecture:			amd64
 Container Runtime Version:	docker://1.12.3
 Kubelet Version:		v1.5.2+43a9be4
 Kube-Proxy Version:		v1.5.2+43a9be4
ExternalID:			192.168.99.100
Non-terminated Pods:		(3 in total)
  Namespace			Name				CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ---------			----				------------	----------	---------------	-------------
  default			docker-registry-1-hfpzp		100m (5%)	0 (0%)		256Mi (12%)	0 (0%)
  default			onspecificnode			0 (0%)		0 (0%)		0 (0%)		0 (0%)
  default			router-1-cdglk			100m (5%)	0 (0%)		256Mi (12%)	0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.
  CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ------------	----------	---------------	-------------
  200m (10%)	0 (0%)		512Mi (25%)	0 (0%)
No events.
```

なお、affinity を使って [node に pod を割り当てる](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)など、上に示したよりも洗練されたやり方があります。ユースケースによっては他のやり方も調べるとよいでしょう。

[前へ](/ic) | [次へ](/api)
