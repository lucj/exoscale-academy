---
title: Exercise
weight: 2
---

In this exercise, you'll persist the VotingApp's databases.

By default, k3s comes with the default StorageClass named *local-path* which can be verified using the following command.

```bash
$ kubectl get storageclass
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  1h4m
```

1. In a file named *pvc-redis.yaml*, define the specification for a *PersistentVolumeClaim* with the following characteristics:

- name: redis
- ReadWriteOnce mode
- request for 1G storage

Then modify the *redis* Deployment specification by adding a volume based on this *PersistentVolumeClaim*, and use the *volumeMounts* instruction to ensure that the associated *PersistentVolume* is mounted in the */data* directory of the redis container.

2. In a file named *pvc-db.yaml* containing the specification for a *PersistentVolumeClaim* with the following characteristics:  

    - name: db
    - ReadWriteOnce mode
    - request for 1G of storage

    Then modify the *db* Deployment specification by adding a volume based on this *PersistentVolumeClaim*, and use the *volumeMounts* instruction to ensure that the associated *PersistentVolume* is mounted in the */var/lib/postgresql/data* directory of the postgres container.

3. Deploy the application defined in these specifications and verify that it is working correctly.

4. List the *PersistentVolumeClaim* resources. What do you observe?

5. Delete the application.
