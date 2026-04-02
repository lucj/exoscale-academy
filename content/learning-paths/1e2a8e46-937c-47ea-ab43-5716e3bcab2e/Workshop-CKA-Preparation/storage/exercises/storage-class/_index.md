---
weight: 3
---

## Exercise

In this exercise, you'll create the simplest possible StorageClass, based on hostPath, for your kubeadm cluster and use it to provision storage.

1. Check the existing StorageClasses

2. Create a hostPath-based StorageClass

3. Create a PersistentVolume manually 

4. Create a PersistentVolumeClaim using your StorageClass

5. Create a Pod that uses the PVC and write some data to the volume

6. Verify the data persists by deleting and recreating the Pod

7. Clean up all resources

## Documentation

[https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

[https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Check the existing StorageClasses

```bash
kubectl get storageclass
kubectl get sc -o wide
```

You should see no StorageClasses:

```bash
No resources found
```

2. Create a hostPath-based StorageClass

Since we don't have a dynamic provisioner, we'll create a simple StorageClass that we can use with manually created PersistentVolumes:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: manual-hostpath
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: false
reclaimPolicy: Delete
EOF
```

Verify the StorageClass was created:

```bash
kubectl get storageclass manual-hostpath
```

{{< hextra/callout type="info" >}}
We use `kubernetes.io/no-provisioner` since we don't have a dynamic provisioner. This means we need to create PersistentVolumes manually. In a real world scenario, the StorageClass triggers the creation of PersistentVolumes.
{{< /hextra/callout >}}

3. Create a PersistentVolume manually

Let's create the following PersistentVolume:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv-001
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: manual-hostpath
  hostPath:
    path: /tmp/k8s-storage
    type: DirectoryOrCreate
EOF
```

Check the PV status:

```bash
kubectl get pv manual-pv-001
```

The PV should be in "Available" status.

4. Create a PersistentVolumeClaim using your StorageClass

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual-hostpath
  resources:
    requests:
      storage: 1Gi
EOF
```

Check the PVC status:

```bash
kubectl get pvc manual-pvc
```

{{< hextra/callout type="warning" >}}
The PVC will remain in "Pending" status until a Pod uses it, due to the `WaitForFirstConsumer` volume binding mode.
{{< /hextra/callout >}}

5. Create a Pod that uses the PVC and write some data to the volume

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  containers:
  - name: demo
    image: alpine:3.22
    command:
      - sleep
      - "3600"
    volumeMounts:
    - name: storage-volume
      mountPath: /data
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: manual-pvc
EOF
```

Wait for the Pod to be running:

```bash
kubectl wait --for=condition=Ready pod/demo --timeout=60s
```

Check that the PVC and the PV are now bound:

```bash
kubectl get pvc manual-pvc
kubectl get pv manual-pv-001
```

Write some data to the volume:

```bash
kubectl exec demo -- sh -c "echo Hello from manual hostPath storage > /data/test.txt"
kubectl exec demo -- sh -c "date >> /data/test.txt"
kubectl exec demo -- sh -c "hostname >> /data/test.txt"
kubectl exec demo -- cat /data/test.txt
```

Check which Node the Pod is running on:

```bash
kubectl get pod demo -o wide
```

6. Verify the data persists by deleting and recreating the Pod

First, delete the demo Pod.

```bash
kubectl delete pod demo
```

Next, recreate the Pod using the same PVC.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: demo2
spec:
  containers:
  - name: demo
    image: alpine:3.22
    command:
      - sleep
      - "3600"
    volumeMounts:
    - name: storage-volume
      mountPath: /data
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: manual-pvc
EOF
```

Wait for the new Pod to be ready and verify the data persists:

```bash
kubectl wait --for=condition=Ready pod/demo2 --timeout=60s
kubectl exec demo2 -- cat /data/test.txt
```

You should see the same data that was written by the first Pod.

{{< hextra/callout type="info" >}}
The second Pod is automatically scheduled on the same Node as the first Pod due to the existing PVC/PV binding. Since we're using hostPath storage, once the PVC is bound to a PV on a specific Node, all subsequent Pods using that PVC will automatically run on the same Node. If you SSH to the Node where the Pods are running, you'll see the data in /tmp/k8s-storage.
{{< /hextra/callout >}}

7. Clean up all resources

```bash
kubectl delete pod demo2
kubectl delete pvc manual-pvc
kubectl delete pv manual-pv-001
kubectl delete storageclass manual-hostpath
```

{{< hextra/callout type="info" >}}
This exercise demonstrates the basic concepts of StorageClass, PV, and PVC without requiring a complex storage provisioner. In a production kubeadm cluster, you would typically use:
- Dynamic provisioners like local-path-provisioner, NFS, ...
- Cloud provider CSI drivers (if running on cloud infrastructure)
- Network-attached storage solutions
{{< /hextra/callout >}}

{{< hextra/callout type="warning" >}}
hostPath storage limitations:
- Data tied to a specific Node
- No replication or high availability
- Manual PV creation required
- Node failure means data loss
- Not suitable for production workloads
{{< /hextra/callout >}}

</details>