---
title: Endpoints
---

## Exercise

1. Create a deployment named *ghost* with 3 pods based on the ghost:4 image

2. Expose the deployment with a ClusterIP service

{{< hextra/callout type="info" >}}
ghost application listens on port 2368
{{< /hextra/callout >}}

3. From the service, retrieved the IP addresses of the pods

4. Inspect the *Endpoints* resource created

5. Delete the deployment and service

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a deployment with 3 pods based on the ghost:4 image

```bash
k create deploy ghost --image=ghost:4 --replicas=3
```

2. Expose the deployment with a ClusterIP service

```bash
k expose deploy/ghost --port 2368
```

3. From the service, retreived the IP addresses of the pods

The pods IP addresses are listed in the Endpoints property:

```bash
k describe svc/ghost
Name:              ghost
Namespace:         default
Labels:            app=ghost
Annotations:       <none>
Selector:          app=ghost
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.110.190.83
IPs:               10.110.190.83
Port:              <unset>  2368/TCP
TargetPort:        2368/TCP
Endpoints:         10.32.0.3:2368,10.38.0.3:2368,10.38.0.5:2368   <- pods' IP addresses
Session Affinity:  None
Events:            <none>
```

4. Inspect the *Endpoints* resource created

When creating the service a Endpoints resource have been created as well.
It lists the IP:PORT of pods in the Ready status and can be described like any other Kubernetes resources:

```bash
k describe  Endpoints ghost
Name:         ghost
Namespace:    default
Labels:       app=ghost
Annotations:  <none>
Subsets:
  Addresses:          10.32.0.3,10.38.0.3,10.38.0.5
  NotReadyAddresses:  <none>
  Ports:
    Name     Port  Protocol
    ----     ----  --------
    <unset>  2368  TCP

Events:  <none>
```

5. Delete the deployment and service

```bash
k delete deploy/ghost svc/ghost
```
</details>

