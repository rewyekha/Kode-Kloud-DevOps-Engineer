# Day 48: Deploy Pods in Kubernetes Cluster

The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

1. Create a pod named `pod-nginx` using the `nginx` image with the `latest` tag. Ensure to specify the tag as `nginx:latest`.
2. Set the `app` label to `nginx_app`, and name the container as `nginx-container`.

`Note`: The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



## Kubernetes Pod Creation – nginx

### Overview

This document explains how to create a Kubernetes Pod using the `kubectl` CLI.\
The task was to create a pod named **pod-nginx** using the **nginx:latest** image with a specific label and container name.

#### Requirements

* Pod name: `pod-nginx`
* Image: `nginx:latest`
* Container name: `nginx-container`
* Label: `app=nginx_app`

***

## Step 1 – Create the Pod

### Command

```bash
kubectl run pod-nginx \
  --image=nginx:latest \
  --labels=app=nginx_app \
  --restart=Never \
  --overrides='{
    "spec": {
      "containers": [{
        "name": "nginx-container",
        "image": "nginx:latest"
      }]
    }
  }'
```

### Terminal Output

```bash
thor@jump-host ~$ kubectl run pod-nginx \
  --image=nginx:latest \
  --labels=app=nginx_app \
  --restart=Never \
  --overrides='{
    "spec": {
      "containers": [{
        "name": "nginx-container",
        "image": "nginx:latest"
      }]
    }
  }'
pod/pod-nginx created
```

### Command Explanation

| Parameter                   | Meaning                                                 |
| --------------------------- | ------------------------------------------------------- |
| `kubectl run`               | Creates a new pod or deployment in Kubernetes           |
| `pod-nginx`                 | Name of the pod                                         |
| `--image=nginx:latest`      | Specifies the container image and tag                   |
| `--labels=app=nginx_app`    | Adds a label to the pod for identification and grouping |
| `--restart=Never`           | Ensures a Pod is created instead of a Deployment        |
| `--overrides`               | Allows manual customization of the pod specification    |
| `"name": "nginx-container"` | Sets the container name inside the pod                  |

***

## Step 2 – Verify Pod Creation

### Command

```bash
kubectl get pods
```

### Terminal Output

```bash
thor@jump-host ~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          7s
```

### Command Explanation

| Column     | Meaning                                       |
| ---------- | --------------------------------------------- |
| `NAME`     | Name of the pod                               |
| `READY`    | Number of ready containers / total containers |
| `STATUS`   | Current state of the pod                      |
| `RESTARTS` | Number of container restarts                  |
| `AGE`      | Time since pod creation                       |

***

## Step 3 – Inspect Pod Details

### Command

```bash
kubectl describe pod pod-nginx
```

### Terminal Output (Important Sections)

```bash
Name:             pod-nginx
Namespace:        default
Node:             jump-host/10.244.164.39
Start Time:       Fri, 13 Mar 2026 03:41:34 +0000
Labels:           app=nginx_app
Status:           Running
IP:               10.22.0.9
```

#### Container Information

```bash
Containers:
  nginx-container:
    Image:          nginx:latest
    State:          Running
    Ready:          True
    Restart Count:  0
```

#### Events

```bash
Events:
  Type    Reason     Message
  ----    ------     -------
  Normal  Scheduled  Successfully assigned default/pod-nginx to jump-host
  Normal  Pulling    Pulling image "nginx:latest"
  Normal  Pulled     Successfully pulled image
  Normal  Created    Created container: nginx-container
  Normal  Started    Started container nginx-container
```

### Command Explanation

| Field        | Meaning                                        |
| ------------ | ---------------------------------------------- |
| `Namespace`  | Kubernetes namespace where the pod runs        |
| `Node`       | Worker node where the pod is scheduled         |
| `Labels`     | Key-value metadata used for grouping resources |
| `Status`     | Current state of the pod                       |
| `Containers` | List of containers inside the pod              |
| `Events`     | Timeline of actions performed by Kubernetes    |

***

## Key Kubernetes Concepts

### Pod

A **Pod** is the smallest deployable unit in Kubernetes and represents one or more containers running together.

### Label

Labels are metadata used for:

* Service discovery
* Resource grouping
* Pod selection

Example:

```
app=nginx_app
```

### Container Image

The container runs using the image:

```
nginx:latest
```

Where:

* `nginx` → image name
* `latest` → tag/version

***

## Quick Validation Checklist

| Check                | Command                                   |
| -------------------- | ----------------------------------------- |
| Verify pod exists    | `kubectl get pods`                        |
| Check labels         | `kubectl get pod pod-nginx --show-labels` |
| Inspect full details | `kubectl describe pod pod-nginx`          |

***

## Useful Cleanup Command

To delete the pod when it is no longer needed:

```bash
kubectl delete pod pod-nginx
```

***

## Summary

We successfully:

1. Created a pod named `pod-nginx`
2. Used image `nginx:latest`
3. Set label `app=nginx_app`
4. Named the container `nginx-container`
5. Verified the pod status and configuration

The pod is running successfully inside the Kubernetes cluster.

```bash
thor@jump-host ~$ kubectl run pod-nginx \
  --image=nginx:latest \
  --labels=app=nginx_app \
  --restart=Never \
  --overrides='{
    "spec": {
      "containers": [{
        "name": "nginx-container",
        "image": "nginx:latest"
      }]
    }
  }'
pod/pod-nginx created
thor@jump-host ~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          7s
thor@jump-host ~$ kubectl describe pod pod-nginx
Name:             pod-nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             jump-host/10.244.164.39
Start Time:       Fri, 13 Mar 2026 03:41:34 +0000
Labels:           app=nginx_app
Annotations:      <none>
Status:           Running
IP:               10.22.0.9
IPs:
  IP:  10.22.0.9
Containers:
  nginx-container:
    Container ID:   containerd://3c36a73d1775aee2fc857bff8361949ed1e7ba841d37d3ec06d2659860e1cfb9
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:bc45d248c4e1d1709321de61566eb2b64d4f0e32765239d66573666be7f13349
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 13 Mar 2026 03:41:38 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kj2gd (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-kj2gd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  17s   default-scheduler  Successfully assigned default/pod-nginx to jump-host
  Normal  Pulling    16s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     13s   kubelet            Successfully pulled image "nginx:latest" in 3.679s (3.679s including waiting). Image size: 62960551 bytes.
  Normal  Created    13s   kubelet            Created container: nginx-container
  Normal  Started    13s   kubelet            Started container nginx-container
thor@jump-host ~$ 
```

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
