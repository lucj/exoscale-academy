---
weight: 1
title: "Challenge 1"
---

Use the following command to break something in the cluster

```bash
curl -s https://learn.exoscale.dev/cka/ch/set1.sh  | bash
```

You should diagnose that the kubectl command to get the nodes, the pods or any other Kubernetes resources does not work anymore. Understand the origin of the problem and fix it.

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Test

If you try to list the Nodes, you'll get an error similar to the following one:

```bash
$ kubectl get no

The connection to the server 192.168.64.17:6443 was refused - did you specify the right host or port?
```

2. Identification

As kubectl targets the API Server, first make sure this one is running fine.

On the controlplane Node, check the APIServer's logs in */var/log/syslog*. You should see that the API Server cannot start correctly:

```bash
/# sudo cat /var/logs/syslog | grep kube-apiserver

2025-08-06T08:38:22.980225+00:00 controlplane kubelet[28526]: E0806 10:38:22.980181   28526 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"StartContainer\" for \"kube-apiserver\" with CrashLoopBackOff: \"back-off 2m40s restarting failed container=kube-apiserver pod=kube-apiserver-controlplane_kube-system(bc6d1be5c0eea4c705d74622b0e5ab0b)\"" pod="kube-system/kube-apiserver-controlplane" podUID="bc6d1be5c0eea4c705d74622b0e5ab0b"
```

Going one step further checking the log of the API Server container in */var/log/containers* shows the configuration of the API Server contains an incorrect property:

```bash
/# sudo cat kube-apiserver-controlplane_kube-system_kube-apiserver-e3de6d80128340cb34a788eb84318e38c16a134e8c637e310531a3426ad03bd4.log 

2025-08-06T10:38:13.342405728+02:00 stderr F I0806 08:38:13.341625       1 options.go:238] external host was not specified, using 192.168.64.17
2025-08-06T10:38:13.34267452+02:00 stderr F E0806 08:38:13.342314       1 run.go:72] "command failed" err="enable-admission-plugins plugin \"DUMMYPLUGIN\" is unknown"

```

Checking the API Server manifest (*/etc*) we can see the DUMMYPLUGIN is there:

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.62.50.215:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - command:
        - kube-apiserver
        - --advertise-address=10.62.50.215
        - --allow-privileged=true
        - --authorization-mode=Node,RBAC
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --enable-admission-plugins=NodeRestriction,DUMMYPLUGIN   <- wrong pluging
        - --enable-bootstrap-token-auth=true
        - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
...
```

3. Fix

Let's remove this plugin from the *--enable-admission-plugins* flag. The API Server should restart correctly after a few tens of seconds (you can restart *kubelet* if this is not the case)

```bash
$ kubectl get no
NAME      STATUS   ROLES                  AGE     VERSION
master    Ready    control-plane,master   2d16h   v1.32.7
worker1   Ready    <none>                 2d16h   v1.32.7
worker2   Ready    <none>                 2d16h   v1.32.7
```

</details>

