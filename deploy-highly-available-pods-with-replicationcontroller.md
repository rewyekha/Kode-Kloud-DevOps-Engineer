# Deploy Highly Available Pods with ReplicationController

The Nautilus DevOps team is establishing a `ReplicationController` to deploy multiple pods for hosting applications that require a highly available infrastructure. Follow the specifications below to create the `ReplicationController`:

1. Create a `ReplicationController` using the `nginx` image with `latest` tag, and name it `nginx-replicationcontroller`.
2. Assign labels `app` as `nginx_app`, and `type` as `front-end`. Ensure the container is named `nginx-container` and set the replica count to `3`.

All `pods` should be running state post-deployment.

`Note:` The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.

```bash
thor@jumphost ~$ kubectl get nodes
NAME                      STATUS   ROLES           AGE   VERSION
kodekloud-control-plane   Ready    control-plane   25m   v1.27.16-1+f5da3b717fc217
thor@jumphost ~$ vi nginx-rc.yaml
thor@jumphost ~$ kubectl apply -f nginx-rc.yaml
replicationcontroller/nginx-replicationcontroller created
thor@jumphost ~$ kubectl get rc nginx-replicationcontroller
NAME                          DESIRED   CURRENT   READY   AGE
nginx-replicationcontroller   3         3         3       9s
thor@jumphost ~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-replicationcontroller-8qspf   1/1     Running   0          15s
nginx-replicationcontroller-k9txn   1/1     Running   0          15s
nginx-replicationcontroller-xw7hb   1/1     Running   0          15s
thor@jumphost ~$ kubectl get pods --show-labels
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
nginx-replicationcontroller-8qspf   1/1     Running   0          26s   app=nginx_app,type=front-end
nginx-replicationcontroller-k9txn   1/1     Running   0          26s   app=nginx_app,type=front-end
nginx-replicationcontroller-xw7hb   1/1     Running   0          26s   app=nginx_app,type=front-end
thor@jumphost ~$ kubectl get nodes
NAME                      STATUS   ROLES           AGE   VERSION
kodekloud-control-plane   Ready    control-plane   27m   v1.27.16-1+f5da3b717fc217
thor@jumphost ~$ 
```

## 🚀 Deploy Highly Available Pods Using ReplicationController

### 📘 Overview

In this task, the Nautilus DevOps team required a **highly available application deployment** using a **ReplicationController**.\
The goal was to ensure that multiple identical Pods are always running to provide fault tolerance and availability.

A ReplicationController was created to manage **three replicas** of an NGINX-based application.

***

### 🎯 Task Requirements

The ReplicationController must meet the following specifications:

* **Name**: `nginx-replicationcontroller`
* **Image**: `nginx:latest`
* **Replica count**: `3`
* **Container name**: `nginx-container`
* **Labels**:
  * `app: nginx_app`
  * `type: front-end`
* **Expected state**: All Pods must be in **Running** state

> ℹ️ Note\
> The `kubectl` utility on `jump_host` is pre-configured to communicate with the Kubernetes cluster.

***

### 🔍 Step 1: Verify Cluster Status

Before creating any resources, confirm that the Kubernetes node is ready.

```bash
kubectl get nodes
```

```
NAME                      STATUS   ROLES           AGE   VERSION
kodekloud-control-plane   Ready    control-plane   25m   v1.27.16-1+f5da3b717fc217
```

✅ Node is in **Ready** state.

***

### 🛠️ Step 2: Create the ReplicationController Manifest

Create the YAML file for the ReplicationController:

```bash
vi nginx-rc.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-replicationcontroller
  labels:
    app: nginx_app
    type: front-end
spec:
  replicas: 3
  selector:
    app: nginx_app
    type: front-end
  template:
    metadata:
      labels:
        app: nginx_app
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

Save and exit the editor.

***

### 🚀 Step 3: Create the ReplicationController

Apply the manifest to the cluster:

```bash
kubectl apply -f nginx-rc.yaml
```

```
replicationcontroller/nginx-replicationcontroller created
```

***

### 🔎 Step 4: Verify the ReplicationController

Check the status of the ReplicationController:

```bash
kubectl get rc nginx-replicationcontroller
```

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-replicationcontroller   3         3         3       9s
```

✅ Desired, current, and ready replicas match.

***

### ✅ Step 5: Verify Pod Status

List all Pods created by the ReplicationController:

```bash
kubectl get pods
```

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-replicationcontroller-8qspf   1/1     Running   0          15s
nginx-replicationcontroller-k9txn   1/1     Running   0          15s
nginx-replicationcontroller-xw7hb   1/1     Running   0          15s
```

✅ All Pods are in **Running** state.

***

### 🏷️ Step 6: Verify Labels (Important for Validation)

Confirm that the Pods have the correct labels:

```bash
kubectl get pods --show-labels
```

```
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
nginx-replicationcontroller-8qspf   1/1     Running   0          26s   app=nginx_app,type=front-end
nginx-replicationcontroller-k9txn   1/1     Running   0          26s   app=nginx_app,type=front-end
nginx-replicationcontroller-xw7hb   1/1     Running   0          26s   app=nginx_app,type=front-end
```

✅ Labels match the selector exactly.

***

### 🔁 Final Cluster Check

```bash
kubectl get nodes
```

```
NAME                      STATUS   ROLES           AGE   VERSION
kodekloud-control-plane   Ready    control-plane   27m   v1.27.16-1+f5da3b717fc217
```

***

### 🎉 Final Outcome

✔ ReplicationController created successfully\
✔ Replica count set to **3**\
✔ Image used: **nginx:latest**\
✔ Container name: **nginx-container**\
✔ Correct labels applied\
✔ All Pods running and healthy

***

### 🧠 Key Learnings & Exam Tips

* ReplicationController ensures a fixed number of Pods are always running
* **Selectors must exactly match Pod template labels**
* Even though ReplicaSets are newer, **ReplicationControllers are still tested**
* Deleting a Pod managed by RC automatically triggers recreation

***

### 📌 One-Line Memory Tip

> **ReplicationController = replicas + selector + pod template**
