---
title: RBAC
---

## Exercise

In this exercise, you'll manage RBAC rules based on the role of your team's members.

Let's assume:
- You're an admin
- A team has the following members: Alice, Bob, Charlie, Dave. Each with different responsibilities

1. Create the Namespaces and RBAC resources

The following table lists the role of each team member

|User|Role|Access|
|----|----|------|
|Alice|Developer|Full access to dev namespace|
|Bob|QA/Tester|Can view & list pods in dev|
|Charlie|DevOps|Can deploy applications in staging|
|Dave|Security Analyst|Read-only to all namespaces|

- Create the ?amespaces: dev, staging
- Create the Roles and RoleBindings for each user
- Simulate access using impersonation

2. Adding Another Developer (Eve)

Eve joins as Developer. Modify the corresponding RoleBinding to include Eve as well.

3. DevOps Needs Access to Production Temporarily

Charlie (DevOps) needs to roll back production.

- Grant access to prod namespace, with time-based audit note
- Create a separate RoleBinding with expiration annotation
- Add a label or an annotation for easy audit

4. Revoke Access for Dave

Dave moves to a different team. Remove his RoleBindings.

## Documentation

[https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Create the Namespaces and RBAC resources

- Creation of the .amespaces

```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod
```

- Creation of a Role for Alice (full access to dev Namespace):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: developer
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
EOF
```

- Creation of a RoleBinding for Alice:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-developer
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
EOF
```

- Creation of a Role for Bob (view Pods in dev):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: qa-tester
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log", "pods/status"]
  verbs: ["get", "list"]
EOF
```

- Creation of a RoleBinding for Bob:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bob-qa
  namespace: dev
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: qa-tester
  apiGroup: rbac.authorization.k8s.io
EOF
```

- Creation of a Role for Charlie (can deploy applications in staging):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: staging
  name: devops
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulset", "daemonset"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete", "scale"]
EOF
```

- Creation of a RoleBinding for Charlie:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: charlie-devops
  namespace: staging
subjects:
- kind: User
  name: charlie
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: devops
  apiGroup: rbac.authorization.k8s.io
EOF
```

- Creation of a ClusterRole for Dave (read access to all namespaces):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: security-analyst
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
EOF
```

- Creation of a ClusterRoleBinding for Dave:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dave-security-analyst
subjects:
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: security-analyst
  apiGroup: rbac.authorization.k8s.io
EOF
```

Use the following commands to test the access using impersonation:

- Testing Alice's access (should work)

```bash
kubectl auth can-i create pods --namespace=dev --as=alice
kubectl auth can-i delete deployments --namespace=dev --as=alice
```

- Testing Bob's access (should only view pods)

```bash
kubectl auth can-i get pods --namespace=dev --as=bob
kubectl auth can-i create pods --namespace=dev --as=bob # Should not be authorized
```

- Testing Charlie's access (can deploy applications in staging)

```bash
kubectl auth can-i create deployments --namespace=staging --as=charlie
kubectl auth can-i create deployments --namespace=dev --as=charlie # Should not be authorized
kubectl auth can-i create configmaps --namespace=dev --as=charlie # Should not be authorized
```

- Tetsing Dave's access (read-only in all Namespaces)

```bash
kubectl auth can-i get pods --namespace=dev --as=dave
kubectl auth can-i get pods --namespace=staging --as=dave
kubectl auth can-i delete pods --namespace=dev --as=dave # Should not be authorized
```

2. Adding Another Developer (Eve)

Modify the existing RoleBinding to include Eve:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-eve-developers
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
- kind: User
  name: eve
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
EOF
```

Testing Eve's access:

```bash
kubectl auth can-i create pods --namespace=dev --as=eve
kubectl auth can-i delete deployments --namespace=dev --as=eve
kubectl auth can-i create pods --namespace=staging --as=eve # Should not be authorized
```

3. DevOps Needs Access to Production Temporarily

Creation of the devops role in prod namespace:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: prod
  name: devops
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete", "scale"]
EOF
```

First, we define an expiration date. The command is different if you're using MacOS or a Linux machine.

```bash
expires_at=$(if [[ "$OSTYPE" == "darwin"* ]]; then
  date -u -v+1d '+%Y%m%dT%H%M%S'
else
  date -u -d '+1 day' '+%Y%m%dT%H%M%S'
fi)
```

{{< hextra/callout type="note" >}}
There is no native process in Kubernetes which automatically removes a resource based on a TTL related annotation. External controller like [k8s-ttl-controller](https://github.com/TwiN/k8s-ttl-controller )could be considered.
{{< /hextra/callout >}}

Next, we create a RoleBinding for Charlie in prod Namespace. We're using a specific annotation to keep track of the expiration date that we could verify in an external script.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: charlie-prod-temp
  namespace: prod
  labels:
    access-type: temporary
    user: charlie
  annotations:
    expires-at: "${expires_at}"
subjects:
- kind: User
  name: charlie
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: devops
  apiGroup: rbac.authorization.k8s.io
EOF
```

Then, we test Charlie's access to the prod Namespace:

```bash
kubectl auth can-i create deployments --namespace=prod --as=charlie
```

The following are example commands to audit this temporary access:

- Listing all temporary access bindings

```bash
kubectl get rolebindings -A -l access-type=temporary
```

- Checking Charlie's specific access

```bash
kubectl get rolebindings -A -l user=charlie
```

{{< hextra/callout type="warning" >}}
In a real world environment, we must have a strong expiration checker process to ensure access are deleted when they are flagged as expired.
{{< /hextra/callout >}}

4. Revoking Access

First, we search for Dave in RoleBindings (there should not be any):

```bash
kubectl get rolebindings -A -o json | jq -r '.items[] | select(.subjects[]?.name == "dave") | "\(.metadata.namespace)/\(.metadata.name)"'
```

Next, we search for Dave in ClusterRoleBindings:

```bash
kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.subjects[]?.name == "dave") | .metadata.name'
```

Next, we remove Dave's access:

```bash
kubectl delete clusterrolebinding dave-security-analyst
```

Then, we verify Dave's access is revoked:

```bash
kubectl auth can-i get pods --namespace=dev --as=dave # Should not be authorized
kubectl auth can-i get deployment --namespace=staging --as=dave # Should not be authorized
```

</details>