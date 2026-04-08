---
title: Gateway
---

## Exercise

0. Prerequisites

{{< hextra/callout type="info" >}}
This exercise assumes you're running on a multipass VM with a 3-node Kubernetes cluster. We'll use NodePort to access the Gateway API controller.
{{< /hextra/callout >}}

Make sure you have installed the [helm client](https://helm.sh/docs/intro/install/) on your host machine.

1. Install the Gateway API CRDs and Envoy Gateway controller

{{< hextra/callout type="info" >}}
We'll use [Envoy Gateway](https://gateway.envoyproxy.io/), which is one of the most mature Gateway API implementations
{{< /hextra/callout >}}

```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.4.2 -n envoy-gateway-system --create-namespace
```

Wait for Envoy Gateway to be ready.

```bash
kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available
```

Create the GatewayClass.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
EOF
```

2. Create a pod named *ghost* based on the ghost:4 image and expose it with a ClusterIP service

{{< hextra/callout type="info" >}}
ghost application listens on port 2368
{{< /hextra/callout >}}

3. Create a Gateway resource

4. Create an HTTPRoute resource to expose the ghost service

5. Verify you can access the ghost web interface

6. Create a self-signed TLS certificate for the ghost application

7. Update the Gateway to support HTTPS with TLS

8. Verify HTTPS access through the Gateway API

9. Clean up all resources

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/gateway/](https://kubernetes.io/docs/concepts/services-networking/gateway/)

[https://gateway-api.sigs.k8s.io/](https://gateway-api.sigs.k8s.io/)

## Solution

1. Install the Gateway API CRDs and Envoy Gateway controller

Commands provided in instructions.

2. Create a pod named *ghost* based on the ghost:4 image and expose it with a ClusterIP service

```bash
kubectl run ghost --image=ghost:4 --port 2368 --expose
```

3. Create a Gateway resource

```bash
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ghost
spec:
  gatewayClassName: eg
  listeners:
  - name: http
    port: 80
    protocol: HTTP
EOF
```

The Gateway will create a LoadBalancer service.

```bash
$ kubectl get svc -n envoy-gateway-system -l gateway.envoyproxy.io/owning-gateway-name=ghost
NAME                           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
envoy-default-ghost-c7075eaa   LoadBalancer   10.96.231.63   <pending>     80:31283/TCP   2m38s
```

The following command gets the value of the NodePort assigned to the HTTP (80) port.

```bash
HTTP_NODEPORT=$(kubectl get svc -n envoy-gateway-system -l gateway.envoyproxy.io/owning-gateway-name=ghost -o jsonpath='{.items[0].spec.ports[?(@.port==80)].nodePort}')
echo "Gateway HTTP accessible on NodePort: $HTTP_NODEPORT"
```

{{< hextra/callout type="warning" >}}
As we'll use the NodePort access, we need to patch the Service so we can reach the Pods from any node.

```bash
kubectl -n envoy-gateway-system patch $(kubectl get svc -n envoy-gateway-system -l gateway.envoyproxy.io/owning-gateway-name=ghost -o name) -p '{"spec":{"externalTrafficPolicy":"Cluster"}}'
```
{{< /hextra/callout >}}


4. Create an HTTPRoute resource to expose the ghost service

We create the following HTTPRoute (this is the Gateway API equivalent of an Ingress resource).

```bash
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: ghost
spec:
  parentRefs:
  - name: ghost
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: ghost
      port: 2368
EOF
```

5. Verify you can access the ghost web interface

```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')

# Get the NodePort (if not already set from previous step)
HTTP_NODEPORT=$(kubectl -n envoy-gateway-system get svc -l gateway.envoyproxy.io/owning-gateway-name=ghost -o jsonpath='{.items[0].spec.ports[?(@.port==80)].nodePort}')

echo "Ghost application accessible at: http://$NODE_IP:$HTTP_NODEPORT"

# Test HTTP access
curl http://$NODE_IP:$HTTP_NODEPORT/
```

The ghost web interface should be accessible at `http://$NODE_IP:$HTTP_NODEPORT`

6. Create a self-signed TLS certificate for the ghost application

```bash
# Generate private key and certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ghost-tls.key \
  -out ghost-tls.crt \
  -subj "/CN=ghost.local"

# Create Kubernetes Secret
kubectl create secret tls ghost-tls-secret \
  --cert=ghost-tls.crt \
  --key=ghost-tls.key
```

7. Update the Gateway to support HTTPS with TLS

```bash
# Update Gateway to include HTTPS listener
cat <<EOF | kubectl apply -f -
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ghost
spec:
  gatewayClassName: eg
  listeners:
  - name: http
    port: 80
    protocol: HTTP
  - name: https
    port: 443
    protocol: HTTPS
    tls:
      mode: Terminate
      certificateRefs:
      - name: ghost-tls-secret
EOF
```

{{< hextra/callout type="warning" >}}
As we'll use the NodePort access, we need to patch the Service so we can reach the Pods from any node.

```bash
kubectl -n envoy-gateway-system patch $(kubectl get svc -n envoy-gateway-system -l gateway.envoyproxy.io/owning-gateway-name=ghost -o name) -p '{"spec":{"externalTrafficPolicy":"Cluster"}}'
```
{{< /hextra/callout >}}

8. Verify HTTPS access through the Gateway API

```bash
# Get the NodePort assigned to HTTPS (port 443)
HTTPS_NODEPORT=$(kubectl -n envoy-gateway-system get svc -l gateway.envoyproxy.io/owning-gateway-name=ghost -o jsonpath='{.items[0].spec.ports[?(@.port==443)].nodePort}')
echo "Gateway HTTPS accessible on NodePort: $HTTPS_NODEPORT"

# Test HTTPS access (with self-signed cert)
curl -k https://$NODE_IP:$HTTPS_NODEPORT/
```

9. Clean up all resources, and remove envoy gateway

```bash
kubectl delete httproute ghost
kubectl delete gateway ghost
kubectl delete secret ghost-tls-secret
kubectl delete pod ghost
kubectl delete service ghost
rm ghost-tls.key ghost-tls.crt
helm uninstall eg -n envoy-gateway-system
```
