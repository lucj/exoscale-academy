---
title: Upgrade
---

## Exercise

In this exercise you will upgrade your cluster to the next minor version.

{{< hextra/callout type="warning" >}}
This exercise is often proposed during the CKA. Make sure to practice it several times.
{{< /hextra/callout >}}

{{< hextra/callout type="info" >}}
You might need to adapt the content if you have a different version of Kubernetes, or a different cluster's composition than the one presented in this exercise.
{{< /hextra/callout >}}


1. Check the version of your cluster

```bash
$ kubectl get no
NAME            STATUS   ROLES           AGE   VERSION
controlplane    Ready    control-plane   32m   v1.32.4
worker1         Ready    <none>          23m   v1.32.4
worker2         Ready    <none>          18m   v1.32.4
```

2. Upgrade the controlplane Node

Run a shell on the controlplane Node and change the file */etc/apt/sources.list.d/kubernetes.list* so it references version *1.33* instead of *1.32*:

Before:
```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /
```

After:
```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /
```

Check the latest version of kubeadm

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

You will be returned the available versions in the *1.33* family. In this exercice we will upgrade to version *1.33.0*, but newer patch versions may exist at the time you are doing it.

First upgrade kubeadm to the new version:

```bash
VERSION=1.33.0-1.1
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next simulate the upgrade

```bash
sudo kubeadm upgrade plan
```

If the previous command went fine we can run the upgrade

```bash
sudo kubeadm upgrade apply v1.33.0
```

Next drain the controlplane Node to prepare it for maintenance

```bash
kubectl drain controlplane --ignore-daemonsets
```

Next upgrade kubelet and kubectl:

```bash
VERSION=1.33.0-1.1
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl
```

Then restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Next uncordon the controlplane Node

```bash
kubectl uncordon controlplane
```

If we check the version, we can see the controlplane Node has been upgraded:

```bash
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   35m    v1.33.0
worker1         Ready    <none>          26m    v1.32.4
worker2         Ready    <none>          21m    v1.32.4
```

{{< hextra/callout type="warning" >}}
If the cluster has several controlplane Nodes they need to be upgraded next. The upgrade command would be slightly different and would look like ```kubeadm upgrade NODE_IDENTIFIER```
{{< /hextra/callout >}}


3. Upgrade the worker Nodes

{{< hextra/callout type="warning" >}}
This procedure shows the upgrade of worker1, the same steps must be done for the other worker Nodes as well.
{{< /hextra/callout >}}

First run a shell on worker1.

As you did for the controlplane Node, change the file */etc/apt/sources.list.d/kubernetes.list* so it references version *1.33* instead of *1.32*.

Next, upgrade the kubeadm binary:

```bash
VERSION=1.33.0-1.1
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next run the upgrade 

```bash
sudo kubeadm upgrade node
```

Next, drain the Node (run the following command from the controlplane Node):

```bash
kubectl drain worker1 --ignore-daemonsets
```

Then, upgrade kubelet and kubectl:

```bash
VERSION=1.33.0-1.1
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl
```

Then restart kubelet

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Next, uncordon the worker Node, making it "schedulable" again (run the following command from the controlplane Node):

```bash
kubectl uncordon worker1
```

If we check the versions, we can see the controlplane Node and the worker1 are now both upgraded:

```bash
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   36m    v1.33.0
worker1         Ready    <none>          27m    v1.33.0
worker2         Ready    <none>          22m    v1.32.4
```

4. Accessing the upgraded cluster

After the above steps are done on the other worker node, the cluster will be fully upgraded cluster:

```bash
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   38m    v1.33.0
worker1         Ready    <none>          29m    v1.33.0
worker2         Ready    <none>          24m    v1.33.0
```