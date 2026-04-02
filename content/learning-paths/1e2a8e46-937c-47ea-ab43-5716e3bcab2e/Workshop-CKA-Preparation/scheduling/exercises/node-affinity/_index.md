## Exercise

1. Create the specification of a Pod named *www* with a single container based on *nginx:1.20*

2. Modify the specification to add a hard constraint forcing the Pod to be deployed on a node with the label *size: large* or *size: medium*

3. Modify the specification to add a soft constraint so the Pod can be deployed on a node with the label *disktype: ssd* if possible

4. Run the Pod. What happens ?

5. Create a label on one of the cluster's nodes so that the pod can be deployed. Make sure the Pod is running fine

6. Delete the pod

## Documentation

[https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a Pod named *www* with a single container based on *nginx:1.20*

```
k run www --image=nginx:1.20 --dry-run=client -o yaml > pod.yaml
```

2. Modify the specification to add a hard constraint forcing the Pod to be deployed on a node with the label *size: large* or *size: medium*

We add the property *.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution* to define the hard constraint that is requested:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: www
  name: www
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - large
            - medium
  containers:
  - image: nginx:1.20
    name: www
```

3. Modify the specification to add a soft constraint so the Pod can be deployed on a node with the label *disktype: ssd* if possible

We add the property *.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution* to define the soft constraint that is requested:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: www
  name: www
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - large
            - medium
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        preference:
          matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - image: nginx:1.20
    name: www
```

4. Run the Pod. What happens ?

Creation of the Pod

```
k apply -f pod.yaml
```

The pod remains *Pending*:

```
k get po
NAME   READY   STATUS    RESTARTS   AGE
www    0/1     Pending   0          26s
```

We can describe the Pod to get additional information on the root cause:

```
k describe po www
...
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  15s (x2 over 100s)  default-scheduler  0/3 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 2 node(s) didn't match Pod's node affinity/selector.
```

The Pod cannot be deployed because there is no node with the label requested by the *hard* constraint

5. Create a label on one of the cluster's nodes so that the Pod can be deployed. Make sure the Pod is running fine

We add a label *size: medium* on worker1

```
k label node worker1 size=medium
```

Note: we could have set the *size* label with the value *medium* or *large* on worker1 or worker2

A couple of second later, the Pod will be deployed on worker1

```
k get po -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP          NODE      NOMINATED NODE   READINESS GATES
www    1/1     Running   0          4m1s   10.32.0.2   worker1   <none>           <none>
```

6. Delete the Pod

```
k delete po www
```

</details>

