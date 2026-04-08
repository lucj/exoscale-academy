---
title: Audit
no_list: true
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
