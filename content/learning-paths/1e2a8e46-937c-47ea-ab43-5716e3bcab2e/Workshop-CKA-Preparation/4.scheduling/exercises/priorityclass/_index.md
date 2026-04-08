---
title: Priority
---

## Exercise

1. Add the label *disktype=ssd* on the worker1 Node

2. Create the specification of a Pod named *nginx* based on the *nginx:1.26* image.

3. Modify the specification making sure the Pod is scheduled on the Node *worker1* and requests 1.5Gi of memory. Then create the resource and verify the Pod is running.

Note: if your Pod stays in *Pending* you can use a lower value for the memory request to make sure it get deployed.

4. Get the Pod's priority and priorityClassName 

5. Create a new PriorityClass named *high* with value *100000*

6. Create a new Pod named *apache* based on the *httpd:2.4* image, making sure it is scheduled on the Node *worker1*, it has the same memory request as the Pod *nginx* and it uses the priorityClass *high*. Once created, get the Pod's priority.

7. What happened ?

8. Delete the Pods, the PriorityClass, and remove the disktype label from worker1 

## Documentation

[https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/#priorityclass)

## Solution

1. Add the label *disktype=ssd* on the worker1 Node

```bash
kubectl label node worker1 disktype=ssd
```

2. Create the specification of a Pod named *nginx* based on the *nginx:1.26* image.

```bash
kubectl run nginx --image=nginx:1.26 --dry-run=client -o yaml > pod.yaml
```

3. Modify the specification making sure the Pod is scheduled on the Node *worker1* and requests 1.5Gi of memory. Then create the resource and verify the Pod is running.

Modification of the specification to add the specific constraints:

```yaml{filename="pod.yaml"}
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - image: nginx:1.26
    name: nginx
    resources:
      requests:
        memory: 1.5Gi
```

Creation of the resource:

```bash
kubectl apply -f pod.yaml
```

Make sure the Pod is running:

```bash
kubectl get po/nginx
```

4. Get the Pod's priority and priorityClassName 

Pod's priority is 0:

```bash
kubectl get po/nginx -o jsonpath={.spec.priority}
0
```

Pod's priorityClassName is not defined

```bash
kubectl get po/nginx -o jsonpath={.spec.priorityClassName}
```

5. Create a new PriorityClass named *high* with value *100000*

```bash
cat <<EOF | kubectl apply -f -
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high
value: 1000000
globalDefault: false
EOF
```

6. Create a new Pod named *apache* based on the *httpd:2.4* image, making sure it is scheduled on the Node *worker1*, it has the same memory request as the Pod *nginx* and it uses the priorityClass *high*. Once created, get the Pod's priority.

```bash
cat<<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: apache
  name: apache
spec:
  priorityClassName: high
  nodeSelector:
    disktype: ssd
  containers:
  - image: httpd:2.4
    name: apache
    resources:
      requests:
        memory: 1.5Gi
EOF
```

Get the Pod's priority:

```bash
kubectl get po/apache -o jsonpath={.spec.priority}
1000000
```

7. What happened ?

Listing the Pods we can see the nginx one is not present anymore:

```bash
kubectl get po
NAME     READY   STATUS    RESTARTS   AGE
apache   1/1     Running   0          14s
```

As the apache Pod has a higher priority than the nginx once, and because the Node worker1 does not have enough resources to run both of them, the nginx Pod has been evicted and replaced by the apache one.

We can see the preemption in the events as well:

```bash
kubectl get events
...
27s         Normal    Preempted          pod/nginx                 Preempted by default/apache on node worker1
```

8. Delete the Pods, the PriorityClass, and remove the disktype label from worker1 

The nginx Pod has already been removed

```bash
kubectl delete po/apache priorityClass/high
kubectl label node worker1 disktype-
```


