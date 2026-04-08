---
title: Ephemeral
---

## Exercise

1. Create a Pod with a single container based on *mongo:5.0* and ensure it persists its data in an ephemeral volume

{{< hextra/callout type="info" >}}
Hint: Mongo persists the data in it's */data/db* folder
{{< /hextra/callout >}}

2. Run a shell in the container and list the content of the */data/db* folder

3. Get the unique identifier of the Pod

{{< hextra/callout type="info" >}}
Hint: This information can be retrieved in the *.metadata.uid* property
{{< /hextra/callout >}}

4. On which node is this Pod running

5. Run a shell on the node this Pod is running on and retrieve the content of the volume

{{< hextra/callout type="info" >}}
Hint: it's located in */var/lib/kubelet/pods/POD_ID/volumes/kubernetes.io~empty-dir/data*
{{< /hextra/callout >}}

6. Delete the Pod

## Documentation

[https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

## Solution

1. Create a Pod with a single container based on *mongo:5.0* and ensure it persists its data in an ephemeral volume

```
kubectl run mongo --image=mongo:5.0 --dry-run=client -o yaml > pod.yaml
```

Add the volume section:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mongo
  name: mongo
spec:
  containers:
  - image: mongo:5.0
    name: mongo
    volumeMounts:
    - name: data
      mountPath: /data/db
  volumes:
  - name: data
    emptyDir: {}
```

Create the Pod:

```
k apply -f pod.yaml
```

2. Run a shell in the container and list the content of the */data/db* folder

```
k exec mongo -- ls /data/db
```

3. Get the unique identifier of the Pod

```
k get po mongo -o jsonpath={.metadata.uid}
```

4. On which node is this Pod running

You can get the information with:

```
k get po mongo -o wide
```

5. Run a shell on the node this Pod is running on and retrieve the content of the volume

Get the content of the following folder, replace POD_ID with the unique identifier you get in question 3.

```
sudo ls /var/lib/kubelet/pods/POD_ID/volumes/kubernetes.io~empty-dir/data
```

6. Delete the Pod

```
k delete po mongo
```


