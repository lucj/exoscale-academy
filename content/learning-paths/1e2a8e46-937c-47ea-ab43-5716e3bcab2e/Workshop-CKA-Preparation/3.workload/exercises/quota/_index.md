---
title: Quota
---

## Exercise

1. Create a Namespace named *dev*

2. Create a ResourceQuota named *limit-pod-number* limiting the number of pods to 5 in that Namespace

3. Create a Deployment named *ghost* with 10 replicas based on ghost:4 in the Namespace *dev*

4. What do you observe ?

5. Get the events which show that some Pods cannot be created

6. Delete the Deployment, the ResourceQuota and the Namespace

## Documentation

[https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

## Solution

1. Create a Namespace named *dev*

```
k create ns dev
```

2. Create a ResourceQuota named *limit-pod-number* limiting the number of Pods to 5 in that Namespace

```
k -n dev create quota limit-pod-number --hard=pods=5
```

3. Create a Deployment named *ghost* with 10 replicas based on ghost:4 in the Namespace *dev*

```
k -n dev create deploy ghost --image=ghost:4 --replicas=10
```

4. What do you observe ?

Only 5 of the 10 replicas are running, the other ones cannot be created in the namespace because of the limitation

```
k -n dev get po -l app=ghost
NAME                     READY   STATUS    RESTARTS   AGE
ghost-5d77b859d5-222q2   1/1     Running   0          26s
ghost-5d77b859d5-96z94   1/1     Running   0          26s
ghost-5d77b859d5-hlbbj   1/1     Running   0          26s
ghost-5d77b859d5-kbdf4   1/1     Running   0          26s
ghost-5d77b859d5-w8rfm   1/1     Running   0          26s
```

5. Get the events which show that some Pods cannot be created

We can get all the events in the *dev* namespace

```
k -n dev get events
```

To get only the one which indicate a creation failure we can add a filter on the *reason* field:

```
k -n dev get events --field-selector reason=FailedCreate
```

6. Delete the Deployment, the ResourceQuota and the Namespace

```
k -n dev delete deploy/ghost quota/limit-pod-number 
k delete ns dev
```


