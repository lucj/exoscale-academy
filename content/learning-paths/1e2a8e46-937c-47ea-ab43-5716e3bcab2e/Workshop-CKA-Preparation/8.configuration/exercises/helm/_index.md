---
title: Helm
---

## Exercise

1. Search for the `nginx` Chart maintained by Bitnami in the ArtifactHub, and find the repository URL

2. Add the Bitnami Helm repository to your local Helm installation

{{< hextra/callout type="info" >}}
The repository's path is https://charts.bitnami.com/bitnami
{{< /hextra/callout >}}

3. Create a namespace called `web-apps`

4. Install the nginx Chart as a release named `my-nginx` in the `web-apps` namespace using version `15.4.4`

5. List all Helm releases and verify the installation status

6. Get the manifest and values of the `my-nginx` release

7. Create a custom values file named `custom-values.yaml` that:
   - Sets the replica count to 3
   - Changes the service type to NodePort
   - Sets a custom image tag to `1.25.3`

8. Upgrade the `my-nginx` release using your custom values file

9. Check the rollout history of the release

10. Rollback the release to the previous version

11. Uninstall the `my-nginx` release

12. Remove the `web-apps` namespace

## Documentation

[https://helm.sh/docs/](https://helm.sh/docs/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Search for the `nginx` Chart maintained by Bitnami in the ArtifactHub, and find the repository URL

```bash
helm search hub nginx bitnami
```

2. Add the Bitnami Helm repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

3. Create a namespace called `web-apps`

```bash
kubectl create namespace web-apps
```

4. Install the nginx Chart as a release named `my-nginx` in the `web-apps` namespace

```bash
helm install my-nginx bitnami/nginx --namespace web-apps
```

{{< hextra/callout type="info" >}}
We can use the `--version` flag to specify the version of the Chart to install. If no version is provided, the latest version of the Chart is installed.
{{< /hextra/callout >}}

5. List all Helm releases and verify the installation status

```bash
helm list --namespace web-apps
helm status my-nginx --namespace web-apps
```

6. Get the manifest and values of the `my-nginx` release

```bash
helm get manifest my-nginx --namespace web-apps
helm get values my-nginx --namespace web-apps
```

7. Create a custom values file named `custom-values.yaml`

```yaml{filename="custom-values.yaml"}
replicaCount: 3

service:
  type: NodePort

image:
  tag: "1.25.3"
```

8. Upgrade the `my-nginx` release using your custom values file

```bash
helm upgrade my-nginx bitnami/nginx --namespace web-apps -f custom-values.yaml
```

9. Check the rollout history of the release

```bash
helm history my-nginx --namespace web-apps
```

10. Rollback the release to the previous version

```bash
helm rollback my-nginx 1 --namespace web-apps
```

11. Uninstall the `my-nginx` release

```bash
helm uninstall my-nginx --namespace web-apps
```

12. Remove the `web-apps` namespace

```bash
kubectl delete namespace web-apps
```

</details>