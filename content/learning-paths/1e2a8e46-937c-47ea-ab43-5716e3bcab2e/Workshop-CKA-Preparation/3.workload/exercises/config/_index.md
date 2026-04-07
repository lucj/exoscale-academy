---
title: configuration
---

## Exercise

1. Create a ConfigMap named mycfg containing the key pairs *token=12345* and *level=debug*

2. Use the *jsonpath* output format to only get the content of the *data* key

3. Create a pod named demo1 with a container based on *nginx:1.20*, making sure each piece of data of the ConfigMap is available in the container as an environment variable of the same name 

4. Run a shell in the nginx container and verify both environment variables are present

5. Create a pod named demo2 with a container based on *nginx:1.20*, make sure the content of the ConfigMap is available in files in */etc/app_cfg*

6. Run a shell in the nginx container of pod demo2 and verify the content of /etc/app_cfg

7. Delete the ConfigMap and the pods

## Documentation

[https://kubernetes.io/docs/concepts/configuration/configmap/](https://kubernetes.io/docs/concepts/configuration/configmap/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a ConfigMap named mycfg containing the key pairs *token=12345* and *level=debug*

```
k create configmap mycfg --from-literal=token=12345 --from-literal=level=debug
```

2. Use the *jsonpath* output format to only get the content of the *data* key

```
k get cm mycfg -o jsonpath={.data}
```

3. Create a pod named demo1 with a container based on *nginx:1.20*, making sure each piece of data of the ConfigMap is available in the container as an environment variable of the same name 

```
k run demo1 --image=nginx:1.20 --dry-run=client -o yaml > demo1.yaml
```

Edit the specification so it defines the environment variables coming from the ConfigMap:

```
apiVersion: v1
kind: pod
metadata:
  labels:
    run: demo1
  name: demo1
spec:
  containers:
  - image: nginx:1.20
    name: demo1
    env:
    - name: token
      valueFrom:
        configMapKeyRef:
          name: mycfg
          key: token
    - name: level
      valueFrom:
        configMapKeyRef:
          name: mycfg
          key: level
```

Create the pod:

```
k apply -f demo1.yaml
```

4. Run a shell in the nginx container and verify both environment variables are present

```
k exec demo1 -- env
...
token=12345
level=debug
```

5. Create a pod named demo2 with a container based on *nginx:1.20*, make sure the content of the ConfigMap is available in files in */etc/app_cfg*

```
k run demo2 --image=nginx:1.20 --dry-run=client -o yaml > demo2.yaml
```

Edit the specification so it defines a volume based on the *mycfg* ConfigMap and mount the content in the nginx container's filesystem:

```
apiVersion: v1
kind: pod
metadata:
  labels:
    run: demo2
  name: demo2
spec:
  containers:
  - image: nginx:1.20
    name: demo2
    volumeMounts:
    - name: cfg
      mountPath: /etc/app_cfg
  volumes:
  - name: cfg
    configMap:
      name: mycfg
```

Create the pod:

```
k apply -f demo2.yaml
```

6. Run a shell in the nginx container of pod demo2 and verify the content of /etc/app_cfg

```
k exec demo2 -- cat /etc/app_cfg/level
debug
```

```
k exec demo2 -- cat /etc/app_cfg/token
12345
```

7. Delete the ConfigMap and the pods

```
k delete cm/mycfg pod/demo1 pod/demo2
```

</details>

