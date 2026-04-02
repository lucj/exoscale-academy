---
weight: 2
---

## Exercise

1. Create the folder */tmp/data* on worker1 

2. Create the specification of a PersistentVolume with the accessMode *ReadWriteOnce*, a storage capacity of 1Gi, a storage of type *hostPath* based on the folder */tmp/data*, and specify the storage class *manual*

3. Modify the specification to make sure this PersistentVolume is related to worker1

{{< hextra/callout type="info" >}}
You can use the nodeAffinity property in the PersistentVolume specification
{{< /hextra/callout >}}

4. Create the PersistentVolume

5. Create a PersistentVolumeClaim with accessMode *ReadWriteOnce* accessMode, a storage capacity of 500Mi and the storage class *manual*

6. Make sure the PVC is bound to the PV

7. Create a Pod named *www*, with a single nginx container based on nginx:1.20. Use the PersistentVolumeClaim as a volume and mount it in */usr/share/nginx/html*. Where is this Pod scheduled ?

8. Create the file *index.html* containing the string *hello* in */tmp/data/* on worker1

9. Send a request to the nginx container and make sure you get the content of the previous file

10. Delete the Pod, the PV and the PVC. Also remove the /tmp/data folder on worker1

## Documentation

- [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the folder */tmp/data* on worker1

From a shell on worker1

```
mkdir /tmp/data
```

2. Create a PersistentVolume with the accessMode *ReadWriteOnce*, a storage capacity of 1Gi, a storage of type *hostPath* based on the folder */tmp/data*, and specify the storage class *manual*


```
cat <<EOF > pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv
spec: 
  storageClassName: "manual"
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
EOF
```

3. Modify the specification to make sure this PersistentVolume is related to worker1

We had a nodeAffinity constraint that links the PersistentVolume to worker1

```
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv
spec: 
  storageClassName: "manual"
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - worker1 
```

4. Create the PersistentVolume

```
k apply -f pv.yaml
```

5. Create a PersistentVolumeClaim with accessMode *ReadWriteOnce* accessMode, a storage capacity of 500Mi and the storage class *manual*

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: pvc
spec: 
  storageClassName: "manual"
  accessModes:
    - ReadWriteOnce
  resources:
    requests: 
      storage: 500Mi
EOF
```

6. Make sure the PVC is bound to the PV

Because the characteristics needed by the PersistentVolumeClaim match the ones offered by the PersistentVolume, the PVC is bound to the PV

```
k get pvc,pv
NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc   Bound    pv       1Gi        RWO            manual         5s

NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pv   1Gi        RWO            Retain           Bound    default/pvc   manual                  49s
```

7. Create a Pod named *www*, with a single nginx container based on nginx:1.20. Use the PersistentVolumeClaim as a volume and mount it in */usr/share/nginx/html*. Where is this pod scheduled ?

Create a Pod specification

```
k run www --image=nginx:1.20 --dry-run=client -o yaml > www.yaml
```

Modification of the specification to define a volume and mount it into the container's filesystem:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: www
  name: www
spec:
  containers:
  - image: nginx:1.20
    name: www
    volumeMounts:
    - name: data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc
```

Creation of the Pod

```
k apply -f www.yaml
```

This Pod is scheduled on worker1, the node associated to the PersistentVolume

```
k get po -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www    1/1     Running   0          4s    10.32.0.2   worker1   <none>           <none>
```

8. Create the file *index.html* containing the string *hello* in */tmp/data/* on worker1

From a shell on worker1

```
echo "hello" | sudo tee /tmp/data/index.html
```

9. Send a request to the nginx container and make sure you get the content of the previous file

```
k exec www -- curl -s localhost
hello
```

10. Delete the Pod, the PV and the PVC. Also remove the /tmp/data folder on worker1

```
k delete po www
k delete pvc/pvc pv/pv
```

From a shell on worker1

```
rm -fr /tmp/data
```

</details>

