---
weight: 7
title: "Challenge 7"
---

Use the following command to break something in the cluster

```bash
curl -s https://learn.exoscale.dev/cka/ch/set7.sh  | bash
```

The pod *demo* does not seem to be running fine and restarts after a few tens of seconds. Try to understand what happens and make sure it is running fine for at least 120 seconds once it is restarted.

<br/>

<details>
  <summary markdown="span">Solution</summary>

Observe the behavior of the pod you will notice the pod is automatically restarted after a few tens of seconds:

```bash
$ kubectl get po -w
NAME   READY   STATUS    RESTARTS   AGE
demo   1/1     Running   0          2s
demo   1/1     Running   1 (1s ago)   76s
```

Going a little deeper in the details, we understand this is due to the liveness probe failing:

```bash
$ kubectl describe po demo
...
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m8s                default-scheduler  Successfully assigned default/demo to worker1
  Normal   Pulled     2m7s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 578.053377ms
  Normal   Pulling    53s (x2 over 2m7s)  kubelet            Pulling image "k8s.gcr.io/busybox"
  Normal   Created    52s (x2 over 2m6s)  kubelet            Created container busybox
  Normal   Started    52s (x2 over 2m6s)  kubelet            Started container busybox
  Normal   Pulled     52s                 kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 512.064103ms
  Warning  Unhealthy  8s (x6 over 93s)    kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    8s (x2 over 83s)    kubelet            Container busybox failed liveness probe, will be restarted
```

If we check the pod's specification, we can see the liveness probe checks the existence of a file:

```bash
$ kubectl get po demo -o yaml
...
apiVersion: v1
kind: Pod
metadata:
  name: demo
  ...
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    image: k8s.gcr.io/busybox
    imagePullPolicy: Always
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      failureThreshold: 3
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1
      timeoutSeconds: 1
```

The */tmp/healthy* file is removed after 30 seconds. According to the liveness probe's specification, 15 seconds after the file is removed the container is considered unhealthy and is restarted.

We can change the command *sleep 30* into *sleep 120* to have an healthy container for a longer period.

First we get the pod specification

```bash
kubectl get po demo -o yaml > demo-pod.yaml
```

Next we change the command:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: demo
  ...
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 120; rm -rf /tmp/healthy; sleep 600        # <- change done here
    image: k8s.gcr.io/busybox
    imagePullPolicy: Always
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      failureThreshold: 3
      initialDelaySeconds: 5
      periodSeconds: 5
      successThreshold: 1
      timeoutSeconds: 1
    ...
```

Then we replace the existing pod:

```bash
cat demo-pod.yaml | kubectl replace --force -f -
pod "demo" deleted
pod/demo replaced
```

The pod should remain healthy for a longer period (more than 2 minutes):

```bash
kubectl get po demo -w
NAME   READY   STATUS    RESTARTS   AGE
demo   1/1     Running   0          6s
demo   1/1     Running   1 (1s ago) 2m46s
```

</details>