---
title: Audit
---

## Exercise

The logging level of audit logs can be set to different levels:

- None: events are not logged
- Metadata: event metadata is logged (requestor, timestamp, resource, verb, etc.), but request and response bodies are not logged
- Request: metadata and the request body are logged, but the response body is not
- RequestResponse: metadata and both request and response bodies are logged

In this exercise, we will consider an audit policy that logs the metadata of each request sent to the API Server. We will use the following configuration file:

```
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

Using the documentation below, configure the API Server to enable audit logging. Make sure to set up a *log backend*. Then create a simple pod and verify that the creation of this Pod is logged in the file containing the audit logs.

## Documentation

[https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)

<br>

<details>
  <summary markdown="span">Solution</summary>

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

</details>