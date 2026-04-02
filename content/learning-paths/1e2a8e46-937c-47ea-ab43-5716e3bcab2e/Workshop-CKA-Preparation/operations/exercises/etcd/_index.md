---
weight: 4
---

In this exercise, you'll create a backup of etcd and restore it.

{{< hextra/callout type="warning" >}}
This exercise is often proposed during the CKA. Make sure to practice it several times.
{{< /hextra/callout >}}

## Exercise

1. Check API Server to etcd communication

From the controlplane node, check the status of etcd

```bash
sudo ETCDCTL_API=3 etcdctl \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
endpoint health
```

You should get a result similar to:

```bash
localhost:2379 is healthy: successfully committed proposal: took = 14.891442ms
```

{{< hextra/callout type="warning" >}}
This demo cluster only uses one etcd instance. In a production cluster at least 3 etcd instances are required, these could run on the control plane Nodes (stacked etcd approach) or on external hosts (external etcd approach)
{{< /hextra/callout >}}

2. Backup etcd

Before creating an etcd backup, create a simple Deployment.

```bash
kubectl create deploy nginx --image=nginx:1.28 --replicas=4
```

Make sure the 4 pods are in the **Running** state.

```bash
k get po -l app=nginx
```

Next, create an etcd backup, this generates the file *snapshot.db* in the current folder:

```bash
sudo ETCDCTL_API=3 etcdctl snapshot save \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
snapshot.db
```

Then, verify the backup file.

```bash
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```

You should get a result similar to:

```bash
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 16d6e9ba |   150574 |       1033 |     7.1 MB |
+----------+----------+------------+------------+
```

3. Restore etcd backup

Before restoring the previous backup, run a new Deployment in the cluster:

```bash
k create deploy mongo --image=mongo:7.0
```

Next, make sure the cluster now contains the 2 Deployments created above:

```bash
k get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo   1/1     1            1           49s
deployment.apps/nginx   4/4     4            4           3m38s

NAME                         READY   STATUS    RESTARTS   AGE
pod/mongo-857d4c74ff-khkbf   1/1     Running   0          49s
pod/nginx-6d777db949-9qg7h   1/1     Running   0          3m38s
pod/nginx-6d777db949-hlb8b   1/1     Running   0          3m38s
pod/nginx-6d777db949-rlzl8   1/1     Running   0          3m38s
pod/nginx-6d777db949-scxt5   1/1     Running   0          3m38s
```

Then restore the backup in the */var/lib/etcd-snapshot* folder (this one will be created automatically during the restoration process):

```bash
sudo ETCDCTL_API=3 etcdctl snapshot restore \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--key=/etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--data-dir /var/lib/etcd-snapshot \
snapshot.db
```

Next, it is necessary to stop the API Server, this can be done by moving the API Server manifests out of the */etc/kubernetest/manifests* folder

```bash
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
```

Next it is necessary to modify the configuration of etcd, so it takes into account the new folder (the one containing the restored backup). Change this path in your etcd manifests (**/etc/kubernetes/manifests/etcd.yaml**), as illustrated in the following example:


```yaml
...
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-snapshot          <- folder to change
      type: DirectoryOrCreate
    name: etcd-data
```

After a few tens of seconds the etcd Pods will be back online with the content of the backup. You can verify etcd is running checking its logs from the folder **/var/log/pods**.

Once etcd is running, restart the API Server moving the specification back in the **/etc/kubernetes/manifests** folder.

```bash
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

Wait for the API Server to be running, checking its logs in **/var/log/pods**.

Then, verify the MongoDB Deployment does not exist anymore in the cluster, as it was created after we created the etcd backup.

```bash
k get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   4/4     4            4           12m
```