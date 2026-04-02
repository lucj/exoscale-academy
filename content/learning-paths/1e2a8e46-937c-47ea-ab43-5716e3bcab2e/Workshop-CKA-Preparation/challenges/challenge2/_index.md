---
weight: 2
title: "Challenge 2"
---

Use the following command to break something in the cluster

```bash
curl -s https://learn.exoscale.dev/cka/ch/set2.sh  | bash
```

The node worker1 seems to have some troubles. Identify the source cause and fix it.

<br/>

<details>
  <summary markdown="span">Solution</summary>

The worker1 has the *NotReady* status

```bash
$ kubectl get no
NAME      STATUS     ROLES                  AGE     VERSION
master    Ready      control-plane,master   2d16h   v1.32.7
worker1   NotReady   <none>                 2d16h   v1.32.7
worker2   Ready      <none>                 2d16h   v1.32.7
```

The description of the node provide additional information

```bash
$ kubectl describe node worker1
...
Events:
  Type    Reason                   Age                From             Message
  ----    ------                   ----               ----             -------
  Normal  RegisteredNode           57m                node-controller  Node worker1 event: Registered Node worker1 in Controller
  Normal  NodeNotReady             25m                node-controller  Node worker1 status is now: NodeNotReady
  Normal  Starting                 24m                kubelet          Starting kubelet.
  Normal  NodeReady                24m                kubelet          Node worker1 status is now: NodeReady
  Normal  NodeAllocatableEnforced  24m                kubelet          Updated Node Allocatable limit across pods
  Normal  NodeHasSufficientMemory  24m (x2 over 24m)  kubelet          Node worker1 status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    24m (x2 over 24m)  kubelet          Node worker1 status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     24m (x2 over 24m)  kubelet          Node worker1 status is now: NodeHasSufficientPID
  Normal  RegisteredNode           22m                node-controller  Node worker1 event: Registered Node worker1 in Controller
  Normal  RegisteredNode           96s                node-controller  Node worker1 event: Registered Node worker1 in Controller
  Normal  NodeNotReady             26s                node-controller  Node worker1 status is now: NodeNotReady
```

The Node in `NodeNotReady` status may indicate kubelet is having trouble. We have the confirmation checking the kubelet status with systemctl on this specific Node.

```bash
sudo systemctl status kubelet
...
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Wed 2025-08-06 10:41:39 CEST; 2min 51s ago
   Duration: 23min 19.123s
       Docs: https://kubernetes.io/docs/
    Process: 170850 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=0/SUCCESS)
   Main PID: 170850 (code=exited, status=0/SUCCESS)
        CPU: 31.077s
```

Let's restart kubelet.

```bash
sudo systemctl start kubelet
```

The worker1 node should appear in the *Ready* status after a few tens of seconds:

```bash
kubectl get no
NAME      STATUS   ROLES                  AGE     VERSION
master    Ready    control-plane,master   2d16h   v1.32.7
worker1   Ready    <none>                 2d16h   v1.32.7
worker2   Ready    <none>                 2d16h   v1.32.7
```

</details>

