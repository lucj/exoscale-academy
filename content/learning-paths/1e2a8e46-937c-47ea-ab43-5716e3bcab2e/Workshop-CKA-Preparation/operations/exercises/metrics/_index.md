---
weight: 5
---

## Exercise

1. Install the metrics server ([https://github.com/kubernetes-sigs/metrics-server](https://github.com/kubernetes-sigs/metrics-server))

2. Verify it's not running correctly

3. Change the configuration adding the *--kubelet-insecure-tls* startup flag

4. Verify it's now running fine

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Install the metrics server ([https://github.com/kubernetes-sigs/metrics-server](https://github.com/kubernetes-sigs/metrics-server))

```
k apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

2. Verify it's not running correctly

Trying to get the Nodes or Pods metrics should return an error

```
k top node
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get nodes.metrics.k8s.io)
```

```
k top pod
Error from server (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
```

Checking the logs of the metrics-server pod will provide additional details

Note: by default the kubelet serving certificate deployed by kubeadm is self-signed ⇒ a connection from external services like the metrics-server to a kubelet cannot be secured with TLS. This is the root cause of the problem we are facing, and the reason why we will use an *insecure* startup option in the next question.


3. Change the configuration adding the *--kubelet-insecure-tls* startup flag

```
k edit deployment.apps/metrics-server -n kube-system
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  ...
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls   <- Adding this flag
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: k8s.gcr.io/metrics-server/metrics-server:v0.6.1
...
```

Note: instead of manually changing the manifest, we can also patch the resource:

```
k patch deployments.apps metrics-server -n kube-system --type=json -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls" }]'
```

4. Verify it's now running fine

After a few tens of seconds, the pods and nodes metrics are available.

```
k top node
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master    160m         8%     1215Mi          64%
worker1   57m          2%     976Mi           51%
worker2   64m          3%     971Mi           51%
```

```
k top pods
NAME    CPU(cores)   MEMORY(bytes)
mongo   5m           68Mi
nginx   0m           3Mi
```

</details>

