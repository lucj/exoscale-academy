---
title: Solution - Audit
---

1. On the *controlplane* node, create the file */etc/kubernetes/audit-policy.yaml* with the following content:

    ```
    apiVersion: audit.k8s.io/v1
    kind: Policy
    rules:
    - level: Metadata
    ```

2. Edit the API Server Pod specification (in the file */etc/kubernetes/manifests/kube-apiserver.yaml*) by adding the following two volume definitions:

    ```
    - name: audit
      hostPath:
        path: /etc/kubernetes/audit-policy.yaml
        type: File
    - name: audit-log
      hostPath:
        path: /var/log/kubernetes/audit/
        type: DirectoryOrCreate
    ```

    and also add the following entries to the *volumeMounts* field of the container:

    ```
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/
      name: audit-log
      readOnly: false
    ```

3. Start a simple Pod:

    ```
    kubectl run www --image=nginx:1.24
    ```

4. Verify that audit logs were generated in the */var/log/kubernetes/audit/* directory on the *controlplane* machine.
