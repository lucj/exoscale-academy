---
weight: 6
---

## Exercise

In this exercise, you'll install [crictl](https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/), a command-line interface for CRI-compatible container runtimes. crictl is an handy tool for low-level troubleshooting when kubectl is not sufficient.

{{< hextra/callout type="warning" >}}
This exercise is an introduction to crictl, so you can manipulate the base commands.
{{< /hextra/callout >}}

1. Install crictl on your worker Nodes

Run the following command on each worker. Make sure to select the right architecture.

```bash
VERSION="v1.33.0"
ARCH=amd64 # Use arm64 if you run your Multipass cluster on MacOS with ARM processor
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-$ARCH.tar.gz
sudo tar zxvf crictl-$VERSION-linux-$ARCH.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-$ARCH.tar.gz

# Verify the installation 
crictl --version
```

2. Configure crictl to connect to the container runtime

3. List and inspect containers using crictl

4. Debug a failing pod using crictl commands

5. Compare crictl output with kubectl output

6. Clean up test resources

## Documentation

[https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/](https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/)

[https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md)

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Install crictl on a worker node

Instructions provided

2. Configure crictl to connect to the container runtime

First, check the container runtime socket (containerd in your environment):

```bash
sudo crictl config --list
```

Then, verify the configuration:

```bash
sudo crictl info
```

3. List and inspect containers using crictl

- Listing all Pods, this is similar to `kubectl get pods` but at container runtime level

```bash
sudo crictl pods
```

- Listing all containers

```bash
sudo crictl ps
```

- Listing all containers including stopped ones

```bash
sudo crictl ps -a
```

- Getting detailed information about containers

```bash
sudo crictl ps -v
```

- listing images present on the current Node

```bash
sudo crictl images
```

- Checking container resources usage

```bash
sudo crictl stats --all
```

4. Debug a failing Pod using crictl commands

First, create the following Pod from your control plane Node:

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: debug-test
  namespace: default
spec:
  containers:
  - name: failing-container
    image: busybox:1.37
    command: ["/bin/sh"]
    args: ["-c", "echo 'Starting...' && sleep 10 && exit 1"]
  - name: working-container
    image: busybox:1.37
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo 'Working...'; sleep 30; done"]
  restartPolicy: Always
EOF
```

Next, check where the Pod is running:

```bash
kubectl get pod debug-test -o wide
```

Next, SSH onto the corresponding worker Node.

Then, use crictl to get the Pod's details and to troubleshoot the error you'll find.

- verify the Pod is on the Node

```bash
sudo crictl pods | grep debug-test
```

- Get the Pod's identifier

```bash
POD_ID=$(sudo crictl pods --name debug-test -q)
```

- Get detailed pod information

```bash
sudo crictl inspectp $POD_ID
```

- List containers in the pod

```bash
sudo crictl ps --pod $POD_ID
```

{{< hextra/callout type="warning" >}}
As this Pod contains a failing container, you may only see the working container in this list.
{{< /hextra/callout >}}

- Get logs from the failing container

```bash
FAILING_CONTAINER_ID=$(sudo crictl ps --pod $POD_ID --name failing-container -aq)
sudo crictl logs $FAILING_CONTAINER_ID
```

{{< hextra/callout type="warning" >}}
-a shows all containers (including stopped), -q shows only IDs
{{< /hextra/callout >}}

You should only see:

```
Starting...
```

- Get logs with timestamps

```bash
sudo crictl logs -t $FAILING_CONTAINER_ID
```

- Follow logs in real-time

```bash
sudo crictl logs -f $FAILING_CONTAINER_ID
```

- Inspect the failing container

```bash
sudo crictl inspect $FAILING_CONTAINER_ID
```

- Execute commands in the working container

```bash
WORKING_CONTAINER_ID=$(sudo crictl ps --pod $POD_ID --name working-container -q)
sudo crictl exec -it $WORKING_CONTAINER_ID /bin/sh
```

{{< hextra/callout type="info" >}}
Type 'exit' to leave the container
{{< /hextra/callout >}}

- Check container statistics

```bash
sudo crictl stats $WORKING_CONTAINER_ID
```

5. Compare crictl output with kubectl output

- From control plane (kubectl):

```bash
kubectl get pods debug-test -o wide
kubectl describe pod debug-test
kubectl logs debug-test -c failing-container
kubectl exec -it debug-test -c working-container -- /bin/sh
```

- From worker node (crictl):

```bash
sudo crictl pods --name debug-test
sudo crictl inspectp $POD_ID
sudo crictl logs $FAILING_CONTAINER_ID
sudo crictl exec -it $WORKING_CONTAINER_ID /bin/sh
```

The key differences between these tools:

- crictl works directly with the container runtime
- crictl shows container runtime IDs vs Kubernetes Pod names
- crictl can access containers even if the kubelet is not responsive
- crictl provides runtime-specific information (cgroup, namespaces, etc.)

6. Clean up test resources

Run the Pod deletion command from the control plane Node:

```bash
kubectl delete pod debug-test
```

From the worker Node, verify the containers and the Pod are gone:

```bash
sudo crictl ps --name debug-test
sudo crictl pods --name debug-test
```

{{< hextra/callout type="warning" >}}
crictl is primarily for debugging and troubleshooting. Always prefer kubectl for normal operations.
{{< /hextra/callout >}}

You'll find below examples of crictl commands for various scenario:

When kubectl is not working, but containers are running:

```bash
sudo crictl ps
sudo crictl logs <container-id>
```

When you need to inspect container mounts and volumes:

```bash
sudo crictl inspect <container-id> | jq '.status.mounts'
```

When debugging network issues at container level:

```bash
sudo crictl inspectp <pod-id> | jq '.status.network'
```

When checking container restart history:

```bash
sudo crictl ps -a | grep <container-name>
```

When you need to stop/start containers manually:

```bash
sudo crictl stop <container-id>
sudo crictl start <container-id>
```

When debugging image pull issues:

```bash
sudo crictl images
sudo crictl pull <image-name>
sudo crictl rmi <image-id>
```

</details>