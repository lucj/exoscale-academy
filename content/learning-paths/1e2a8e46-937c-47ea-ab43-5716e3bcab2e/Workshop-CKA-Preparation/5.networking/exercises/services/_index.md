---
title: Service
---

## Exercise

1. Create a pod named *podinfo* with a single container based on the image stefanprodan/podinfo

2. Expose the pod with a ClusterIP service

{{< hextra/callout type="info" >}}
The podinfo container listens on port 9898
{{< /hextra/callout >}}

3. Get a shell in a temporary pod and make sure you can reach the podinfo service

4. Modify the type of the service from ClusterIP to NodePort

5. Access the application via the NodePort service

6. Delete the service and pod

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

## Solution

1. Create a pod with a single container based on the image stefanprodan/podinfo

```
k run podinfo --image=stefanprodan/podinfo
```

2. Expose the pod with a ClusterIP service

```
k expose pod/podinfo --port 9898
```

3. Get a shell in a temporary pod and make sure you can reach the podinfo service

Creation of a temporary pod

```
k run -ti --rm debug --image=alpine -- sh
```

Access the podinfo service (on its port 9898):

```
/ # wget -qO- podinfo:9898
{
  "hostname": "podinfo",
  "version": "6.1.1",
  "revision": "",
  "color": "#34577c",
  "logo": "https://raw.githubusercontent.com/stefanprodan/podinfo/gh-pages/cuddle_clap.gif",
  "message": "greetings from podinfo v6.1.1",
  "goos": "linux",
  "goarch": "amd64",
  "runtime": "go1.17.8",
  "num_goroutine": "6",
  "num_cpu": "2"
}
```

4. Modify the type of the service from ClusterIP to NodePort

Edit the service

```
k edit svc/podinfo
```

Change the type property from ClusterIP to NodePort

```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2022-04-01T08:22:53Z"
  labels:
    run: podinfo
  name: podinfo
  namespace: default
  resourceVersion: "807726"
  uid: eaca8934-6887-430b-a639-f44a084fb9f5
spec:
  clusterIP: 10.96.181.199
  clusterIPs:
  - 10.96.181.199
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 9898
    protocol: TCP
    targetPort: 9898
  selector:
    run: podinfo
  sessionAffinity: None
  type: NodePort          <-- changing the service type
status:
  loadBalancer: {}
```

5. Access the application via the NodePort service

Get the associated port (31029 in this example)

```
k get svc/podinfo
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
podinfo   NodePort   10.96.181.199   <none>        9898:31029/TCP   26m
```

Using the IP address of one of the cluster's nodes:

```
curl 10.62.50.173:31029
{
  "hostname": "podinfo",
  "version": "6.1.1",
  "revision": "",
  "color": "#34577c",
  "logo": "https://raw.githubusercontent.com/stefanprodan/podinfo/gh-pages/cuddle_clap.gif",
  "message": "greetings from podinfo v6.1.1",
  "goos": "linux",
  "goarch": "amd64",
  "runtime": "go1.17.8",
  "num_goroutine": "6",
  "num_cpu": "2"
}
```

6. Delete the service and pod

```
k delete svc/podinfo po/podinfo
```


