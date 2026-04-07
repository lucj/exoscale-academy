---
title: node
---

## Exercise

This exercise covers comprehensive node maintenance procedures including cordon, drain, and uncordon operations that are critical for CKA exam success.

### Scenario 1: Scheduled Maintenance

1. Create a Deployment named *web-app* with 6 replicas using nginx:1.26 image

2. Create a DaemonSet named *monitoring* using nginx:alpine image

3. Check how Pods are distributed across all Nodes

4. You need to perform maintenance on worker1. First, `cordon`this Node to prevent new Pods from being scheduled on it

5. Verify that worker1 is cordoned, but existing Pods are still running

6. Drain worker1 to move all Pods to other nodes

7. Verify all web-app Pods have been moved out of worker1, while DaemonSet Pods are still running

8. Make worker1 available for scheduling again

9. Force the restart of the web-app deployment and verify some Pods are scheduled on worker1

10. Clean up all resources

## Documentation

- [https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
- [https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration](https://kubernetes.io/docs/concepts/architecture/nodes/#manual-node-administration)

<br/>

<details>
  <summary markdown="span">Solution</summary>

### Scenario 1: Scheduled Maintenance

1. Create a Deployment named *web-app* with 6 replicas using nginx:1.26 image

```bash
kubectl create deployment web-app --image=nginx:1.26 --replicas=6
```

2. Create a DaemonSet named *monitoring* using nginx:alpine image

```bash
kubectl create -f - <<EOF
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
      - name: monitoring
        image: nginx:alpine
        ports:
        - containerPort: 80
EOF
```

3. Check how Pods are distributed across all Nodes

```bash
kubectl get pods -o wide
```

You should see Pods are distributed across worker Nodes, and one monitoring Pod per Node.

4. You need to perform maintenance on worker1. First, `cordon`this Node to prevent new Pods from being scheduled on it

```bash
kubectl cordon worker1
```

5. Verify that worker1 is cordoned, but existing Pods are still running

- worker1 should show as Ready,SchedulingDisabled

```bash
kubectl get nodes
```

- existing Pods on worker1 should still be running

```bash
kubectl get pods -o wide
```

6. Drain worker1 to move all Pods to other nodes

```bash
kubectl drain worker1 --ignore-daemonsets --delete-emptydir-data --force
```

{{< hextra/callout type="warning" >}}
- `--ignore-daemonsets` flag is needed because DaemonSet Pods cannot be moved
- `--delete-emptydir-data` flag ensures that Pods with emptydir volumes can be deleted
- `--force` flag ensures Pods that are not managed by a controller can be destroyed
{{< /hextra/callout >}}

7. Verify all web-app Pods have been moved out of worker1, while DaemonSet Pods are still running

```bash
kubectl get pods -o wide
```

8. Make worker1 available for scheduling again

```bash
kubectl uncordon worker1
```

9. Force the restart of the web-app deployment and verify some Pods are scheduled on worker1

```bash
kubectl rollout restart deployment/web-app
kubectl get pods -o wide
```

You should notice that some Pods are scheduled on worker1.

10. Clean up all resources

```bash
kubectl delete deployment web-app
kubectl delete daemonset monitoring
```

</details>