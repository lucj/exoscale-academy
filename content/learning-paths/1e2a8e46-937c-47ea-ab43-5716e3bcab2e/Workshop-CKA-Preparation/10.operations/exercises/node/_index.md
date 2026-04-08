---
title: Node
no_list: true
---

## Exercise

This exercise covers comprehensive node maintenance procedures including cordon, drain, and uncordon operations that are critical for CKA exam success.

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
