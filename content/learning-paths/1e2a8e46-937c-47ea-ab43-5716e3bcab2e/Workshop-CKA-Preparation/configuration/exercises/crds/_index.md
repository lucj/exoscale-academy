## Exercise

1. Create a CustomResourceDefinition named `applications.company.com` that defines an Application resource with the following properties:  
  
   - Group: `company.com`
   - Kind: `Application`
   - Plural: `applications`
   - Singular: `application`
   - Short names: `app`, `apps`
   - Scope: `Namespaced`  
  
2. The Application CRD should have a schema with these fields under `spec`:

   - `name` (string, required)
   - `version` (string, required)
   - `replicas` (integer, minimum: 1, maximum: 10)
  
3. Apply the CRD to your cluster and verify it's been created
  
4. Check that Kubernetes recognizes the new Application resource type
  
5. Create an Application resource named `webapp` with the following specifications:
   - name: "my-web-app"
   - version: "1.2.0"
   - replicas: 3
  
6. List all Application resources and get the details of the `webapp` resource
  
7. Update the `webapp` resource to change the version to "1.3.0"
  
8. Delete the Application resource and the CRD
  
## Documentation

[https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create a CustomResourceDefinition named `applications.company.com`

This CRD is defined in the following specification.

```yaml{filename="application-crd.yaml"}
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.company.com
spec:
  group: company.com
  scope: Namespaced
  names:
    kind: Application
    plural: applications
    singular: application
    shortNames:
    - app
    - apps
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required:
            - name
            - version
            properties:
              name:
                type: string
              version:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 10
```

2-3. Apply the CRD to your cluster and verify it's been created

```bash
kubectl apply -f application-crd.yaml
kubectl get crd applications.company.com
```

4. Check that Kubernetes recognizes the new Application resource type

```bash
$ k api-resources | grep application
applications                        app,apps     company.com/v1                      true         Application
```

5. Create an Application resource named `webapp`

Create the following specification:

```yaml{filename="webapp.yaml"}
apiVersion: company.com/v1
kind: Application
metadata:
  name: webapp
spec:
  name: "my-web-app"
  version: "1.2.0"
  replicas: 3
```

Apply it:

```bash
$ kubectl apply -f webapp.yaml
application.company.com/webapp created
```

6. List all Application resources and get the details of the `webapp` resource

```bash
kubectl get applications
kubectl get application webapp -o yaml
```

You can also use the shortname `app` as defined in the CRD.:

```bash
kubectl get apps
kubectl describe app webapp
```

7. Update the `webapp` resource to change the version to "1.3.0"

```bash
kubectl patch application webapp --type='merge' -p='{"spec":{"version":"1.3.0"}}'
```

Or you can edit directly:

```bash
kubectl edit application webapp
```

Then, verify the change is taken into account:

```bash
kubectl get application webapp -o yaml
```

8. Delete the Application resource and the CRD

```bash
kubectl delete application webapp
kubectl delete crd applications.company.com
```

</details>