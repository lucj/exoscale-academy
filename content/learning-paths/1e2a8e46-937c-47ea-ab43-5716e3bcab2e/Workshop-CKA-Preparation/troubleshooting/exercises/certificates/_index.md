---
weight: 5
---

## Exercise

1. On a kubeadm cluster, where are located the private keys and related certificates used by the control plane components to communicate with each other ?

2. Where is the certificate used by kubelet to communicate with the API Server ?

3. Using openssl, get the Group and User that identifiy the kubelet agent

4. Get the rights associated to the previous Group

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. On a kubeadm cluster, where are located the private keys and related certificates used by the control plane components to communicate with each other ?

The PKI are located on the master nodes in the folder */etc/kubernetes*

```bash
find /etc/kubernetes
/etc/kubernetes
/etc/kubernetes/kubelet.conf
/etc/kubernetes/controller-manager.conf
/etc/kubernetes/manifests
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/etcd.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/scheduler.conf
/etc/kubernetes/pki
/etc/kubernetes/pki/front-proxy-ca.key
/etc/kubernetes/pki/front-proxy-client.key
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/front-proxy-client.crt
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/etcd
/etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/peer.key
/etc/kubernetes/pki/etcd/healthcheck-client.key
/etc/kubernetes/pki/etcd/healthcheck-client.crt
/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/etcd/peer.crt
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/sa.pub
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/admin.conf
```

Some certificates / keys are under the *pki* subfolder, other ones are defined in the kubeconfig files (.conf extension)

2. Where is the certificate used by kubelet to communicate with the API Server ?

The certificate is defined in the */etc/kubernetes/kubelet.conf* file

```bash
sudo cat /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ETXlOVEUyTURVeU0xb1hEVE15TURNeU1qRTJNRFV5TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT1d0CjYySjErK1pONGRXYm1MZlJIUkZ0aVhoVW5YQ1BRbkdWMFVVbFlTSlFwS1VRSndiVDM0UjQ4bUZ1eVpkWEEzNUEKSGtlVkpSM214RndYVjdQWmtqV2RHRWNsT2hWdFp6UVR4akM0eEwyRkdEL2srNFhJbUdobWJycWRMd3Yzdm1NNwpONFUvcjIrMUQ5MDExdkkvVVlZQXc4Q2xwUll2KzErZkZrZDR1YlRIY3VnMW9sRUR1bTVzekZQaUx1RlRYREJYCjYvWEJhZzdKWU1Wc3d2MHpGa3kxTzJFRFI0NDRCVG9hbGN1djVucGdpdWFnTlptV3laYnNKWURyYlBrV2t6Nk8KZE5ZcmFXTXNSckJRcFgvVjl0NkRuUzc2eVdQS3ZhUUZzbkNLVnFMRzVPWm44c0lKMUprN2JGV2wzVWNsYlIrQwpCdXFwSWZZNFZJVS90ZWpsOTUwQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZLZm9uajAvWURhVjBCZXo2djVXT2xydG1ReWhNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTEtQRlNwNHN4b3ZxTllVY25YVApnbEZ4NkdMMC9STXptM251cTVSSitYdGtqUkZJczhXVERhS0dRaXdRbU56Mm9obFExcHM3MmdJaE5nbzcrYXZtCktudGZCVGpQR2R3YXE3bStGd0c3TGh1Y1pQL0hLb3BqN2dqbllJbFBjcjhiSXRlUi9nUE12NDJQSE93VzVnQkQKdkd2SFJpMXJ6dEdLcnZlS1FPOEpycU44Wm83bzMxc2RrZ20xcTdsNUFERTdxTEJiUzVXQmxtd2hWeE5NQUI4cApqZnpMWFFwK0FWazVzQnJWZzF2c0UzbXR3SzVkR29sQkNaMUxPVGtzVGcrdDA1eW0zOUlFdkRSdnJ2RS9EVzJFCitOY3FOY254RGtHdUVMc2pmUGgzeE9pWUpKelFBNzVhaGFERnAyWFJPUDZjU1FZU3VIM2x1Wk1pOXFBclhPSjUKenhBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://194.182.171.68:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:node:master
  name: system:node:master@kubernetes
current-context: system:node:master@kubernetes
kind: Config
preferences: {}
users:
- name: system:node:master
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
```

In this example, the certificate is located at */var/lib/kubelet/pki/kubelet-client-current.pem*

3. Using openssl, get the Group and User that identify the kubelet agent

Getting the content of the certificate using openssl:

```bash
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout
```

Output (only the key information is presented):

```yml
Certificate:
    ...
    Subject: O = system:nodes, CN = system:node:master
    ...
```

From the subject information, we can get the Group and User which authenticates the kubelet agent:

- Group is *system:nodes*
- User is *system:node:master*

4. Get the rights associated to the previous Group

Getting the ClusterRole with the same name as the Group (*system:node*):

```bash
kubectl get clusterrole system:node -o yaml
```

The rules of this clusterrole are listed under the `rules` property:

```yaml
rules:
- apiGroups: [""]
  resources: ["nodes", "nodes/status"]
  verbs: ["get", "list", "watch", "create", "patch", "update"]
- apiGroups: [""]
  resources: ["pods", "pods/status"]
  verbs: ["get", "list", "watch", "create", "delete", "patch", "update"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch", "update"]
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["certificates.k8s.io"]
  resources: ["certificatesigningrequests"]
  verbs: ["create", "get", "list", "watch"]
  ...
```

{{< hextra/callout type="info" >}}
The kubelet agent is allowed to perform many actions on the cluster including managing Nodes, Pods, and accessing ConfigMaps/Secrets needed for Pod operation.
{{< /hextra/callout >}}

</details>

