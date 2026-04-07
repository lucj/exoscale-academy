---
title: Secret
---

## Exercise

1. Create a file *private.txt* with the content

```
token=cb3456a54EB5
```

2. Create a Secret named *credentials*, of type *generic*, from the file *private.txt*. Make sure the key under the *data* property of the secret is *creds*

3. Create a Pod named *test*, with a single container based on the *alpine* image and running the "sleep 3600" command. Make sure this Pod has access to the content of the Secret in the */secrets/credentials* folder in the container's filesystem

4. Run a shell in the Pod's container and verify the content of the */secrets/credentials/creds*

5. Delete the Pod and the Secret

## Documentation

[https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the file *private.txt*

```
cat >> private.txt << EOF
token=cb3456a54EB5
EOF
```

2. Create a Secret named credentials from this file

```
k create secret generic credentials --from-file=creds=./private.txt
```

The *data* property contains the *creds* key:

```
k get secret credentials -o yaml
apiVersion: v1
data:
  creds: dG9rZW49Y2IzNDU2YTU0RUI1Cg==
kind: Secret
metadata:
  name: credentials
type: Opaque
```

3. Create a Pod named *test*, with a single container based on the *alpine* image and running the "sleep 3600" command. Make sure this Pod has access to the content of the Secret in the */secrets/credentials* folder in the container's filesystem

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: alpine
    name: alpine
    command:
    - "sleep"
    - "3600"
    volumeMounts:
    - name: creds
      mountPath: /secrets/credentials
  volumes:
  - name: creds
    secret:
      secretName: credentials
```

4. Run a shell in the Pod's container and verify the content of the */secrets/credentails/creds*

```
k exec test -- cat /secrets/credentials/creds
token=cb3456a54EB5
```

5. Delete the Pod and the Secret

```
k delete po/test secret/credentials
```
</details>

