---
title: Command
---

## Exercise

1. Create a Pod named *debug* with one container based on *alpine:3.15* and make sure it executes the command `sleep 10000`

2. Run a shell in the container and list the processes running inside of it

3. Delete the Pod

## Documentation

- [https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)
- [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#run)

## Solution

1. Create a Pod named *debug* with one container based on *alpine:3.15* and make sure it executes the command `sleep 10000`

```
k run debug --image=alpine:3.15 --command sleep 10000
```

2. Run a shell in the container and list the processes running inside of it

```
k exec debug -- ps aux
```

3. Delete the Pod

```
k delete po debug
```

