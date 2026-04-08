---
title: Ingress
---

## Exercise

0. Prerequisites

{{< hextra/callout type="info" >}}
This exercise assumes you're running on a multipass VM with a 3-node Kubernetes cluster. We'll use NodePort to access the Ingress controller.
{{< /hextra/callout >}}

Make sure you have installed the [helm client](https://helm.sh/docs/intro/install/) on your host machine.

1. Install the NGinx ingress controller with Helm using NodePort

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress ingress-nginx/ingress-nginx --version 4.13.0 \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=30080 \
  --set controller.service.nodePorts.https=30443
```

2. What is the type of Service exposing the ingress controller and on which ports?

3. Get the Node IP to access the Ingress Controller

```bash
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Ingress accessible at: http://$NODE_IP:30080"
```

Make sure this URL is reachable. It should return a 404 as no Ingress resources have been created yet.

4. Create a pod named *ghost* based on the ghost:4 image and expose it with a ClusterIP service

{{< hextra/callout type="info" >}}
ghost application listens on port 2368
{{< /hextra/callout >}}

5. Create an ingress resource that exposes the ghost service

6. Verify you can access the ghost web interface through the NodePort

7. Delete the ingress resource, the pod and the service, and the ingress controller

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)

## Solution

1. Install the NGinx ingress controller with Helm using NodePort

Command provided in instructions.

2. What is the type of Service exposing the ingress controller and on which ports?

The ingress controller is exposed with a NodePort service on ports 30080 (HTTP) and 30443 (HTTPS):

```bash
$ kubectl get svc ingress-ingress-nginx-controller
NAME                               TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-ingress-nginx-controller   NodePort   10.110.37.224   <none>        80:30080/TCP,443:30443/TCP   114s
```

3. Get the Node IP to access the Ingress Controller

```bash
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Ingress accessible at: http://$NODE_IP:30080"
```

This allows direct access to the Ingress Controller through any node in the cluster.

4. Create a pod named *ghost* based on the ghost:4 image and expose it with a ClusterIP service

```bash
kubectl run ghost --image=ghost:4 --port 2368 --expose
```

5. Create an ingress resource that exposes the ghost service

```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ghost-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ghost
            port:
              number: 2368
EOF
```

6. Verify you can access the ghost web interface through the NodePort

```bash
# Get node IP if not already set
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')

# Test access to ghost application
curl http://$NODE_IP:30080/
```

The ghost web interface should be accessible at `http://$NODE_IP:30080/`

7. Delete the ingress resource, the pod and the service, and the ingress controller

```bash
kubectl delete ingress ghost-ingress
kubectl delete pod ghost
kubectl delete service ghost

helm uninstall ingress
```


