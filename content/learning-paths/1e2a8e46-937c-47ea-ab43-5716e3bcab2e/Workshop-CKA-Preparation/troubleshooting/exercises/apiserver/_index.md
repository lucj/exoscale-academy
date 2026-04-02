---
weight: 6
---

## Exercise

This exercise simulates an API server failure and demonstrates how to troubleshoot it.

1. Check the current status of all control plane components

2. Simulate an API server failure by introducing a configuration error:
   - SSH to the control plane node
   - Edit `/etc/kubernetes/manifests/kube-apiserver.yaml` (you need `sudo` for this)
   - Change the `--etcd-servers` parameter to an invalid value like `https://127.0.0.1:2380` (wrong port)

3. Wait 1-2 minutes and check what happens to the cluster

4. Find the issue and fix the configuration

5. Verify the API server is working again

## Documentation

- [https://kubernetes.io/docs/tasks/debug/debug-cluster/](https://kubernetes.io/docs/tasks/debug/debug-cluster/)

- [https://kubernetes.io/docs/concepts/cluster-administration/logging/](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Check the current status of all control plane components

```bash
kubectl get pods -n kube-system
kubectl get nodes
kubectl get componentstatuses  # this command is deprecated but is still useful
```

2. Simulate API server failure (on control plane node):

Instructions provided above

3. Wait and observe the failure:

Any kubectl command should timeout with an error message.

```bash
$ kubectl get nodes
Get "https://192.168.64.17:6443/api/v1/nodes?limit=500": net/http: TLS handshake timeout - error from a previous attempt: read tcp 192.168.64.17:43892->192.168.64.17:6443: read: connection reset by peer
```

4. Find the issue and fix the configuration

First, check the API server logs on control plane node:

```bash
sudo journalctl -u kubelet | grep apiserver
```

You'll get information similar to:

```bash
Aug 06 11:37:28 controlplane kubelet[28526]: E0806 11:37:28.906487   28526 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"StartContainer\" for \"kube-apiserver\" with CrashLoopBackOff: \"back-off 40s restarting failed container=kube-apiserver pod=kube-apiserver-controlplane_kube-system(985738be376c918fcccfba9135af2a6f)\"" pod="kube-system/kube-apiserver-controlplane" podUID="985738be376c918fcccfba9135af2a6f"
```

This indicates kubelet is not able to start the API Server, but at this stage we don't know why.

Let's check the log of the API Server container from `/var/log/containers` (we need to use `sudo` for that purpose). First, we get the corresponding log file.

```bash
$ cd /var/log/containers
$ ls -al | grep kube-apiserver
```

Next, we can check for errors in this file.

```bash
$ cat kube-apiserver-controlplane_kube-system_kube-apiserver-6171a92ceb23ba2828f84bf3bf875d39ca71272c804606ce52202cbb021b5023.log
2025-08-06T11:40:13.180718092+02:00 stderr F W0806 09:40:13.180666       1 logging.go:55] [core] [Channel #2 SubChannel #4]grpc: addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2380", ServerName: "127.0.0.1:2380", }. Err: connection error: desc = "transport: Error while dialing: dial tcp 127.0.0.1:2380: connect: connection refused"
```

The logs show the API Server cannot connect to etcd. This is due to the incorrect port number, as etcd listens on 2379.

Then, fix the configuration by changing the port from 2380 back to 2379:

```bash
sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
# Change back to: --etcd-servers=https://127.0.0.1:2379
```

5. Verify API server is working:

After a few tens of seconds, the API server will be back to normal.

```bash
$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   5d21h   v1.32.7
worker1        Ready    <none>          5d21h   v1.32.7
worker2        Ready    <none>          5d21h   v1.32.7
```

Below are the key troubleshooting commands we used:

```bash
# Check component status
kubectl get pods -n kube-system
kubectl get nodes

# Check static pod manifests
ls -la /etc/kubernetes/manifests/

# Check API server logs via kubelet (on control plane node)
sudo journalctl -u kubelet | grep apiserver

# Check kubelet logs
sudo journalctl -u kubelet -f

# Check control plane logs in /var/log/containers
sudo cat /var/log/containers/kube-apiserver...
```

</details>
