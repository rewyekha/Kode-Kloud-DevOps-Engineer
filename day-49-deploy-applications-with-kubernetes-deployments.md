# Day 49: Deploy Applications with Kubernetes Deployments
The Nautilus DevOps team is delving into Kubernetes for app management. One team member needs to create a deployment following these details:

Create a deployment named `nginx` to deploy the application `nginx` using the image `nginx:latest` (ensure to specify the tag)

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

### Objective
Deploy an **nginx** application using a Kubernetes **Deployment**.

### Steps
#### 1. Create a Deployment
Run the following command to create a deployment named `nginx` using the `nginx:latest` image:

```bash
kubectl create deployment nginx --image=nginx:latest
```

**Output:**

```
deployment.apps/nginx created
```

***

#### 2. Verify the Deployment
Check the status of the deployment:

```bash
kubectl get deployments
```

**Example Output:**

```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           14s
```

* **READY:** 1/1 → All pods are running
* **UP-TO-DATE:** 1 → Deployment has updated all desired replicas
* **AVAILABLE:** 1 → Number of pods available to serve traffic

***

#### 3. Check Pods
List the pods created by the deployment:

```bash
kubectl get pods
```

**Example Output:**

```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7c5d8bf9f7-htt5v   1/1     Running   0          22s
```

* Each pod name is automatically generated with a **ReplicaSet suffix**.
* **STATUS: Running** confirms the pod is active.

***

#### 4. Describe the Deployment
Get detailed information about the deployment:

```bash
kubectl describe deployment nginx
```

**Key Information:**

* **Name:** nginx
* **Replicas:** 1 desired | 1 updated | 1 available | 0 unavailable
* **StrategyType:** RollingUpdate
* **Pod Template Labels:** app=nginx
* **Container Image:** nginx:latest

**Events:**

```
Scaled up replica set nginx-7c5d8bf9f7 from 0 to 1
```

* Confirms the deployment created a **new ReplicaSet** and scaled pods.

***

#### 5. Summary
* A **Kubernetes Deployment** `nginx` was successfully created.
* The deployment manages **ReplicaSet** and **Pod** automatically.
* Kubernetes ensures **desired replicas are running** and handles updates via rolling strategy.
