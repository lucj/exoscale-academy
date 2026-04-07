---
title: Logs
---

## Exercise

1. Get the kubelet logs on one of your cluster's worker nodes

2. Check the logs of the control-plane components

3. Where are the control-plane log files located ?

## Documentation

[https://kubernetes.io/docs/concepts/cluster-administration/logging/](https://kubernetes.io/docs/concepts/cluster-administration/logging/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Get the kubelet logs on one of your cluster's worker nodes

From a shell on the node:

```
sudo journalctl -u kubelet
```

2. Check the logs of the control-plane components

If your control plane node is named *controlplane*, then you can get the logs using the following commands:

```
# API server
k -n kube-system logs kube-apiserver-controlplane

# Controller Manager
k -n kube-system logs kube-controller-manager-controlplane

# Scheduler
k -n kube-system logs kube-scheduler-controlplane

# etcd
k -n kube-system logs etcd-controlplane
```

3. Where are the control-plane log files located ?

The log files of the control-plane components are located under */var/log/pods/* on the control plane node.

```
ls /var/log/pods
kube-system_kube-apiserver-controlplane_5a5ed5eae21579319207ea468e95b498
kube-system_kube-controller-manager-controlplane_df0779e51a7e9b81028a512e0d0d3f51
kube-system_kube-proxy-qkgmx_fc6ce579-e910-42df-9e63-0a733746df5b
kube-system_kube-scheduler-controlplane_c529a0e58147976e9813a2df72cd8520
...
```

Note: /var/log/pods also contains the log files of any other pods running on the node

</details>