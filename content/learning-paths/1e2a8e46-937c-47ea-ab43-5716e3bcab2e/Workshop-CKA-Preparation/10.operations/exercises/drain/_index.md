---
title: Drain
no_list: true
---

## Exercise

1. Create a Deployment named *www*, with 4 replicas of Pods based on the nginx:1.20 image

2. Where are the Pods scheduled ?

3. Drain the worker1 node. What happened ?

4. Uncordon worker1

5. What happened for the Pods belonging to the www Deployment ?

6. Force the recreation of the Pods. What happened ?

7. Delete the Deployment

## Documentation

[https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)
