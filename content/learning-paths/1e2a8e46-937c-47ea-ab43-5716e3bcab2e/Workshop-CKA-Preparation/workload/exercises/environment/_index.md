## Exercise

1. Create a pod named *alpine* with a single container based on *alpine:3.15*. This container must run the command `sleep 3600` and have the environment variable *token=12345*

2. Verify by running a shell in the container and displaying the environment. The variable named *token* should be present.

3. Delete the pod

## Documentation

[https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a pod with a single container based on *alpine:3.15*. This container must run the command `sleep 3600` and have the environment variable *token=12345*

```
k run alpine --image=alpine:3.15 --env token=12345 --command sleep 3600
```

2. Verify by running a shell in the container and displaying the environment. The variable named *token* should be present.

```
k exec alpine -- env
```

3. Delete the pod

```
k delete po/alpine
```

</details>

