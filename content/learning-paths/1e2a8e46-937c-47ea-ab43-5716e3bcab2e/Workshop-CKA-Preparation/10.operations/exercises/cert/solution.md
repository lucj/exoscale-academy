---
title: Solution
---

1. Check the current certificate expiration dates

Run the following command from the control plane node.

```bash
sudo kubeadm certs check-expiration
```

This command shows all certificates managed by kubeadm and their expiration dates:

- admin.conf (client certificate for kubectl)
- apiserver (API server serving certificate)
- apiserver-etcd-client (API server to etcd client certificate)
- apiserver-kubelet-client (API server to kubelet client certificate)
- controller-manager.conf (controller manager client certificate)
- etcd-healthcheck-client (etcd health check client certificate)
- etcd-peer (etcd peer certificate for cluster communication)
- etcd-server (etcd server certificate)
- front-proxy-client (front proxy client certificate)
- scheduler.conf (scheduler client certificate)
- super-admin.conf (client certificate bypassing authorization layer)

{{< hextra/callout type="warning" >}}
- By default, kubeadm certificates are valid for 1 year. You should renew them before they expire to avoid cluster outages.
- Certificate are automatically renewed during each upgrade of the control plane.
{{< /hextra/callout >}}


2. Renew the certificates using kubeadm

You can renew all certificates at once.

```bash
sudo kubeadm certs renew all
```

Or, you can renew specific certificates, for example:

```bash
sudo kubeadm certs renew apiserver
sudo kubeadm certs renew apiserver-kubelet-client
sudo kubeadm certs renew controller-manager.conf
sudo kubeadm certs renew scheduler.conf
sudo kubeadm certs renew admin.conf
sudo kubeadm certs renew super-admin.conf
```

Once, you have renewed the certificates, you can verify the expiration dates have been updated.

```bash
sudo kubeadm certs check-expiration
```

3. Restart the control plane components

First, restart kubelet.
```bash
sudo systemctl restart kubelet
```

Next, move the static Pods manifests temporarily to stop the processes.

```bash
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
sudo mv /etc/kubernetes/manifests/kube-controller-manager.yaml /tmp/
sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /tmp/
sudo mv /etc/kubernetes/manifests/etcd.yaml /tmp/
```

Next, wait a couple of seconds to ensure the control plane Pods are terminated

Next, move the manifests back to restart the control plane Pods

```bash
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
sudo mv /tmp/kube-controller-manager.yaml /etc/kubernetes/manifests/
sudo mv /tmp/kube-scheduler.yaml /etc/kubernetes/manifests/
sudo mv /tmp/etcd.yaml /etc/kubernetes/manifests/
```

Then, wait a few tens of seconds for the Pods to restart.

4. Verify the cluster is working correctly

First, update your local kubeconfig with the renewed admin certificate

```bash
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

Next, check the cluster's status.

```bash
kubectl cluster-info
```

You can also verify the Nodes are ready, and run a couple of basic commands.

```bash
kubectl get nodes
kubectl get pods -n kube-system
kubectl get namespaces
```

{{< hextra/callout type="info" >}}
For each worker Node, the kubelet certificate is automatically renewed by kubelet when it's about to expire.
{{< /hextra/callout >}}
