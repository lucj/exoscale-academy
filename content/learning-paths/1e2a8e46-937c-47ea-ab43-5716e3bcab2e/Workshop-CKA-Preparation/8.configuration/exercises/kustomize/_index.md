---
title: Kustomize
---

## Exercise

1. Create a directory structure for a Kustomize project with:
   - a base layer
   - a development and a production overlays

2. Create resources in the base layer:
   - A Deployment named `webapp` using `nginx:1.26` image with 2 replicas
   - A Service named `webapp-service` exposing port 80

3. Create a `kustomization.yaml` file in the base directory that references both resources

4. Apply the base configuration

5. Configure the `development` overlay:
   - Changes the image to `nginx:1.28`
   - Adds a `env: development` label to all resources
   - Sets the replica count to 1

6. Preview the development overlay using `kubectl kustomize` without applying

7. Apply the development overlay

8. Create a `production` overlay that:
   - Changes the image to `nginx:1.28-alpine`
   - Adds a `env: production` label to all resources
   - Sets the replica count to 5
   - Changes the service type to LoadBalancer

9. Preview the production overlay

10. Apply the production overlay

11. Verify the production changes

12. Delete the resources

## Documentation

[https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/](https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/)

## Solution

1. Create a directory structure for the Kustomize project

```bash
mkdir -p base
mkdir -p overlays/development
mkdir -p overlays/production
```

2. Create resources in the base layer:

Create the following specifications in the base folder:

{{< hextra/tabs items="deployment.yaml,service.yaml" >}}

  {{< hextra/tab >}}
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: webapp
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: webapp
    template:
      metadata:
        labels:
          app: webapp
      spec:
        containers:
        - name: webapp
          image: nginx:1.26
          ports:
          - containerPort: 80
  ```
  {{< /hextra/tab >}}
  
  {{< hextra/tab >}}
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: webapp-service
  spec:
    selector:
      app: webapp
    ports:
    - port: 80
      targetPort: 80
    type: ClusterIP
  ```
  {{< /hextra/tab >}}
  
{{< /hextra/tabs >}}


3. Create a `kustomization.yaml` file in the base directory that references both resources

```yaml{filename="kustomization.yaml"}
resources:
- deployment.yaml
- service.yaml
```

4. Apply the base configuration

```bash
$ kubectl apply -k base
service/webapp-service created
deployment.apps/webapp created
```

5. Configure the `development` overlay:

In the `overlays/development` folder, create the following `kustomization.yaml` file:

```yaml{filename="kustomization.yaml"}
resources:
- ../../base

labels:
- pairs:
    env: development

patches:
- patch: |-
    - op: replace
      path: /spec/template/spec/containers/0/image
      value: nginx:1.28
    - op: replace
      path: /spec/replicas
      value: 1
  target:
    kind: Deployment
    name: webapp
```

This file specifies:
- the base resources to use
- the labels to add
- the patches that need to be applied to the base resources to change the image and the number of replicas

{{< hextra/callout type="info" >}}
Instead of defining the patches in the kustomization file, we could also create the `deployment-patch.yaml` file in the `overlays/development`:

```yaml{filename="deployment-patch.yaml"}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: webapp
        image: nginx:1.28
```

And update the kustomization.yaml, so it applies this patch:

```yaml{filename="kustomization.yaml"}
resources:
- ../../base

labels:
- pairs:
    env: development

patches:
- deployment-patch.yaml
```
{{< /hextra/callout >}}

6. Preview the development overlay:

kubectl has the `kustomize` built-in command for that purpose:

```bash
kubectl kustomize overlays/development
```

You should see the resulting Deployment's specification is now based on `nginx:1.28` and has 1 replica.

7. Apply the development overlay:

```bash
$ k apply -k overlays/development           
service/webapp-service configured
deployment.apps/webapp configured
```

8. Configure the production overlay

In the `overlays/production` folder, create the following `kustomization.yaml` file:

```yaml{filename="kustomization.yaml"}
resources:
- ../../base

labels:
- pairs:
    env: production

patches:
- patch: |-
    - op: replace
      path: /spec/template/spec/containers/0/image
      value: nginx:1.28-alpine
    - op: replace
      path: /spec/replicas
      value: 5
  target:
    kind: Deployment
    name: webapp
- patch: |-
    - op: replace
      path: /spec/type
      value: LoadBalancer
  target:
    kind: Service
    name: webapp-service
```

{{< hextra/callout type="info" >}}
As we've explained for the development overlay, we could also create external patches for both the Deployment and the Service instead of specifying the patches operation in the `kustomization.yaml`
{{< /hextra/callout >}}

9. Preview the production overlay:

```bash
kubectl kustomize overlays/production
```

10. Apply the production overlay:

```bash
kubectl apply -k overlays/production
```

11. Verify the production changes:

{{< hextra/callout type="info" >}}
Since both overlays target the same resources, only the last applied configuration (production) is visible in the cluster. In real scenarios, you would deploy overlays to different namespaces.
{{< /hextra/callout >}}

Checking deployment:

```bash
kubectl get deployment webapp -o wide
kubectl get pods -l app=webapp
```

You should see 5 replicas running nginx:1.28-alpine.

Checking service:

```bash
kubectl get service webapp-service
```

You should see the Service type is now LoadBalancer.

Checking labels:

```bash
kubectl get deployment webapp --show-labels
kubectl get service webapp-service --show-labels
```

You should see the `env: production` label applied to both the Deployment and the Service.

12. Clean up:

```bash
kubectl delete -k overlays/production
```

{{< hextra/callout type="info" >}}
This kustomization.yaml is a bit tricky; you'll not have to create one from scratch in the CKA, but you need to understand Kustomize's main concepts
{{< /hextra/callout >}}
