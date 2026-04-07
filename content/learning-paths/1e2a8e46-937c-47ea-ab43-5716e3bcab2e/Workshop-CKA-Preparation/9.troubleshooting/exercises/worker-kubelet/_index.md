---
title: kubelet
---

## Exercise

1. Verify that all your worker nodes are healthy

2. Stop the kubelet process on the worker1

3. Check the status of worker1

4. Restart the kubelet process on worker1 and make sure everything is fine

<br/>

<details>
  <summary markdown="span">Solution</summary>

1. Verify that all your worker nodes are healthy

List the nodes and make sure they all have the status Ready

```
k get no
```

Describe the worker nodes to get additional information

```
k describe node worker1
```

2. Stop the kubelet process on the worker1

First run a shell on worker1, then stop kubelet (it is managed by *systemd*)

```
sudo systemctl stop kubelet
```

Make sure it is correctly stopped

```
sudo systemctl status kubelet
```

3. Check the status of worker1

From the list of node, worker1 should appear as NotReady

```
k get no
```

Describing worker1 yoy should see the node as an unknown status (*NodeStatusUnknown*)

```
k describe node worker1
```

4. Restart the kubelet process on worker1 and make sure everything is now fine

First run a shell on worker1, then start kubelet

```
sudo systemctl start kubelet
```

Make sure it is correctly started

```
sudo systemctl status kubelet
```

Listing the node or describing *worker1* you should see everything is back to normal

</details>

