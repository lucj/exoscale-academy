---
title: Rollout and Rollback
---

## Exercise

1. Create a deployment named *www* with 5 replicas of *nginx:1.16*

2. Update the deployment so it uses *nginx:1.18* and check the rollout status

3. Update the deployment so it uses *nginx:1.20* and check the rollout status

4. Get the history of the rollout

5. Rollback the deployment to the first revision

6. Verify the image tag of the nginx pods is 1.16

7. Delete the deployment

## Documentation

[https://kubernetes.io/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a deployment named *www* with 5 replicas of *nginx:1.16*

Creation of the deployment:

```
k create deploy www --replicas=5 --image=nginx:1.16
```

Making sure the pods are running fine:

```
k get po -l app=www
NAME                   READY   STATUS    RESTARTS   AGE
www-785d86969c-84g25   1/1     Running   0          34s
www-785d86969c-brrm8   1/1     Running   0          34s
www-785d86969c-l8l6b   1/1     Running   0          34s
www-785d86969c-sjnxw   1/1     Running   0          34s
www-785d86969c-zdgpt   1/1     Running   0          34s
```

2. Update the deployment so it uses *nginx:1.18* and check the rollout status

Updating the image:

```
k set image deploy/www nginx=nginx:1.18
```

Checking the rollout status

```
k rollout status deploy/www
deployment "www" successfully rolled out
```

3. Update the deployment so it uses *nginx:1.20* and check the rollout status

Updating the image:

```
k set image deploy/www nginx=nginx:1.20
```

Checking the rollout status

```
k rollout status deploy/www
deployment "www" successfully rolled out
```

4. Get the history of the rollout

The following command gets the deployment's history:

```
k rollout history deploy/www
deployment.apps/www
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
```

5. Rollback the deployment to the first revision

Rolling back:

```
k rollout undo --to-revision=1 deploy/www
```

Making sure the rollback was succesful:

```
k rollout status deploy/www
deployment "www" successfully rolled out
```

6. Verify the image tag of the nginx pods is 1.16

```
k get po -l app=www -o jsonpath='{range .items[*]}{.spec.containers[0].image}{"\n"}{end}'
nginx:1.16
nginx:1.16
nginx:1.16
nginx:1.16
nginx:1.16
```

7. Delete the deployment

```
k delete deploy/www
```
</details>

