---
title: Roles
---

## Exercise

1. Make sure user *thomas* can list the Pods cluster wide

2. Make sure user *thomas* can create a port-forward on all the Pods in the *dev* Namespace 

3. Make sure user *thomas* can create, list, get, update, delete the Deployments in the *dev* Namespace

4. Make sure user *patrick* can manage (all actions) the Deployment named *www* in the *dev* Namespace

5. Delete the Role / ClusterRole / RoleBinding / ClusterRoleBinding created as well as the *dev* Namespace.

## Documentation

- [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [https://kubernetes.io/docs/reference/kubectl/generated/kubectl_auth/kubectl_auth_can-i/](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_auth/kubectl_auth_can-i/)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Make sure user *thomas* can list the Pods cluster wide

Start by creating a ClusterRole allowing to list the Pods in the entire cluster:

```
k create clusterrole list-pods --verb list --resource pods 
```

Associate the *ClusterRole* to *thomas* via a *ClusterRoleBinding*

```
k create clusterrolebinding thomas-list-pods --clusterrole list-pods --user thomas
```

Verify:

```
k auth can-i list pods --as thomas
yes
```

2. Make sure user *thomas* can create a port-forward on all the Pods in the *dev* Namespace 

First create the *dev* namespace

```
k create ns dev
```

Create the Role:

```
k create role port-forward --verb create --resource pods/forward --namespace dev 
```

Associate the *Role* to *thomas* via a *RoleBinding*

```
k create rolebinding thomas-port-forward --role port-forward --user thomas --namespace dev
```

Verify:

```
k auth can-i create pods --subresource=forward --as thomas --namespace dev 
yes
```

3. Make user user *thomas* can create, list, get, update, delete the Deployments in the *dev* Namespace

Create the Role:

```
k create role manage-deployment --verb create,list,get,update,delete --resource deployments.apps --namespace dev 
```

Associate the *Role* to *thomas* via a *RoleBinding*

```
k create rolebinding thomas-manage-deployment --role manage-deployment --user thomas --namespace dev
```

Verify:

```
k auth can-i create deployments.apps --as thomas --namespace dev
yes
```

4. Make sure user *patrick* can manage (all actions) the Deployment named *www* in the *dev* Namespace

Create a role that allows to manage the deployment named *www*:

```
k create role manage-www-deployment --verb="*" --resource=deployment.apps --resource-name=www -n dev
```

Associate that role to user *patrick*:

```
k create rolebinding patrick-manage-www-deployment --user=patrick --role=manage-www-deployment -n dev
```

Verify:

```
k auth can-i "*" deploy/www --as patrick -n dev
yes
```

5. Delete the Role / ClusterRole / RoleBinding / ClusterRoleBinding created as well as the *dev* namespace.

```
k delete rolebinding patrick-manage-www-deployment thomas-manage-deployment thomas-port-forward
k delete role manage-www-deployment manage-deployment port-forward 
k delete clusterrolebinding thomas-list-nodes
k delete clusterrole list-nodes
k delete ns dev
```

</details>

