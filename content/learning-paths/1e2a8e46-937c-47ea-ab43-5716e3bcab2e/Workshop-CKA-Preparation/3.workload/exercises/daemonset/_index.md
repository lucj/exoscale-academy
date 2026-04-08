---
title: DaemonSet
---

## Exercise

1. Create the specification of a DaemonSet named *log*. Each Pod must:
- be based on the alpine:3.15 image
- read the log files located in */var/log/pods* on the node it is running on
- stream the log on its standard output

Note: path of the log files to stream is /var/log/pods/\*/\*/\*.log

2. Create the DaemonSet. Where are the Pods scheduled ? 

3. Change the specification so there is a Pod on each node of the cluster

4. Verify the Pod can stream the node's log

5. Delete the DaemonSet

## Documentation

- [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

## Solution

1. Create the specification of a DaemonSet named *log*. Each Pod must:
- be based on the alpine:3.15 image
- read the log files located in */var/log/pods* on the node it is running on
- stream the log on its standard output

As it is not possible to create a DaemonSet directly with kubectl, we start by creating a Deployment

```
k create deploy log --image=alpine:3.15 --dry-run=client -o yaml > spec.yaml
```

Then we modify the Deployment to change it into a DaemonSet:
- modification of the *kind* from Deployment to DaemonSet
- removal of the *replicas* and *strategy* properties

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
```

Next we add a volume which gives access to the */var/log/pods* folder of the node

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

Then we mount the volume into the filesystem of the alpine container in */var/log/pods*

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
  
```

Finally we define the command which reads the logs and stream them:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
        command:
        - "/bin/sh"
        - "-c"
        - "tail -f /var/log/pods/*/*/*.log"
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

2. Create the DaemonSet. Where are the Pods scheduled ? 

Creation of the DaemonSet

```
k apply -f spec.yaml
```

Listing the Pods, we can see there is no Pod on the controlplane node:

```
k get po -o wide
log-2tzzd   1/1     Running       0          17s   10.38.0.3   worker2   <none>           <none>
log-smtmn   1/1     Running       0          17s   10.32.0.4   worker1   <none>           <none>
```

This is due to the controlplane's taint that the Pod does not tolerate. The command below shows the key and effect of that taint (you might need to install jq if it's not already on your machine):

```
k get no controlplane -o jsonpath={.spec.taints} | jq
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/control-plane"
  }
]
```

3. Change the specification so there is a Pod on each node of the cluster

We add the toleration so a Pod of the DaemonSet can also be scheduled on the controlplane node:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - image: alpine:3.15
        name: alpine
        command:
        - "/bin/sh"
        - "-c"
        - "tail -f /var/log/pods/*/*/*.log"
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

Applying the new specification

```
k apply -f spec.yaml
```

There is now one Pod per node:

```
k get po -o wide
log-2l2zz   1/1     Running   0          72s   10.40.0.1   controlplane    <none>           <none>
log-676qv   1/1     Running   0          37s   10.32.0.3   worker1         <none>           <none>
log-m8q56   1/1     Running   0          5s    10.38.0.3   worker2         <none>           <none>
```

4. Verify the Pod can stream the node's log

Get the log of one of the DaemonSet's Pod:

```
k logs 2l2zz
```

Or the logs of all DaemonSet Pods at once

```
k logs ds/log
```

5. Delete the DaemonSet

```
k delete ds log
```

