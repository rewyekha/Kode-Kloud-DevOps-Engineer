# Update Deployment and Service in Kubernetes

An application deployed on the Kubernetes cluster requires an update with new features developed by the Nautilus application development team. The existing setup includes a deployment named `nginx-deployment` and a service named `nginx-service`. Below are the necessary changes to be implemented without deleting the deployment and service:

1.) Modify the service nodeport from `30008` to `32165`

2.) Change the replicas count from `1` to `5`

3.) Update the image from `nginx:1.19` to `nginx:latest`

`Note:` The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.

```bash
thor@jumphost ~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-dc49f85cc-ffw7f   1/1     Running   0          76s
thor@jumphost ~$ kubectl get deployment nginx-deployment-dc49f85cc-ffw7f
Error from server (NotFound): deployments.apps "nginx-deployment-dc49f85cc-ffw7f" not found
thor@jumphost ~$ kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           111s
thor@jumphost ~$ kubectl get service nginx-service
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:30008/TCP   2m7s
thor@jumphost ~$ kubectl edit service nginx-service
service/nginx-service edited
thor@jumphost ~$ kubectl get service nginx-service
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:32165/TCP   3m34s
thor@jumphost ~$ kubectl scale deployment nginx-deployment --replicas=5
deployment.apps/nginx-deployment scaled
thor@jumphost ~$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-dc49f85cc-4qd42   1/1     Running   0          12s
nginx-deployment-dc49f85cc-6dr6q   1/1     Running   0          12s
nginx-deployment-dc49f85cc-7htgl   1/1     Running   0          12s
nginx-deployment-dc49f85cc-9k44v   1/1     Running   0          11s
nginx-deployment-dc49f85cc-ffw7f   1/1     Running   0          4m1s
thor@jumphost ~$ kubectl describe deployment nginx-deployment | grep -i image
    Image:         nginx:1.19
thor@jumphost ~$ kubectl set image deployment/nginx-deployment nginx=nginx:latest
error: unable to find container named "nginx"
thor@jumphost ~$ kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Fri, 16 Jan 2026 18:21:52 +0000
Labels:                 app=nginx-app
                        type=front-end
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-app
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-app
  Containers:
   nginx-container:
    Image:         nginx:1.19
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-dc49f85cc (5/5 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  5m17s  deployment-controller  Scaled up replica set nginx-deployment-dc49f85cc to 1
  Normal  ScalingReplicaSet  88s    deployment-controller  Scaled up replica set nginx-deployment-dc49f85cc to 5 from 1
thor@jumphost ~$ kubectl set image deployment/nginx-deployment nginx=nginx:latest
error: unable to find container named "nginx"
thor@jumphost ~$ kubectl set image deployment/nginx-deployment nginx-container=nginx:latest
deployment.apps/nginx-deployment image updated
thor@jumphost ~$ kubectl rollout status deployment/nginx-deployment
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
thor@jumphost ~$ kubectl describe deployment nginx-deployment | grep -i image
    Image:         nginx:latest
thor@jumphost ~$ kubectl get deployment nginx-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           6m36s
thor@jumphost ~$ kubectl get service nginx-service
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:32165/TCP   6m43s
thor@jumphost ~$ 
```

Perfect 👍\
Below is the **complete GitBook-style documentation** for your **KodeKloud challenge**, written cleanly, professionally, and **including all your terminal outputs exactly as requested**.

You can **copy–paste this directly into GitBook** (Markdown-based).

***

## 🚀 KodeKloud Challenge: Updating an Existing Kubernetes Deployment & Service

### 📘 Overview

An application deployed on a Kubernetes cluster required an update with new features developed by the Nautilus application development team.

The existing setup already included:

* A **Deployment** named `nginx-deployment`
* A **Service** named `nginx-service`

The task was to apply the required changes **without deleting** the deployment or the service.

***

### 🎯 Objectives

The following changes were required:

1. **Modify Service NodePort**
   * From `30008` → `32165`
2. **Scale Deployment Replicas**
   * From `1` → `5`
3. **Update Container Image**
   * From `nginx:1.19` → `nginx:latest`

> ⚠️ Constraint\
> The deployment and service **must not be deleted**.

***

### 🔍 Initial Verification

#### Check existing Pods

```bash
kubectl get pods
```

```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-dc49f85cc-ffw7f   1/1     Running   0          76s
```

***

#### Attempting to query Deployment using Pod name (Incorrect)

```bash
kubectl get deployment nginx-deployment-dc49f85cc-ffw7f
```

```
Error from server (NotFound): deployments.apps "nginx-deployment-dc49f85cc-ffw7f" not found
```

✅ **Explanation**\
Pods have autogenerated names. The Deployment name is **not** the same as the Pod name.

***

#### Verify the Deployment

```bash
kubectl get deployment nginx-deployment
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           111s
```

***

#### Verify the Service

```bash
kubectl get service nginx-service
```

```
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:30008/TCP   2m7s
```

***

### 🛠️ Step 1: Update the Service NodePort

Edit the service:

```bash
kubectl edit service nginx-service
```

```
service/nginx-service edited
```

Change:

```yaml
nodePort: 30008
```

To:

```yaml
nodePort: 32165
```

#### Verify the change

```bash
kubectl get service nginx-service
```

```
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:32165/TCP   3m34s
```

✅ **Task 1 completed**

***

### 🔄 Step 2: Scale Deployment Replicas (1 → 5)

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

```
deployment.apps/nginx-deployment scaled
```

#### Verify Pods

```bash
kubectl get pods
```

```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-dc49f85cc-4qd42   1/1     Running   0          12s
nginx-deployment-dc49f85cc-6dr6q   1/1     Running   0          12s
nginx-deployment-dc49f85cc-7htgl   1/1     Running   0          12s
nginx-deployment-dc49f85cc-9k44v   1/1     Running   0          11s
nginx-deployment-dc49f85cc-ffw7f   1/1     Running   0          4m1s
```

✅ **Task 2 completed**

***

### 🖼️ Step 3: Update the Container Image

#### Check current image

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

```
Image:         nginx:1.19
```

***

#### First (incorrect) attempt

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:latest
```

```
error: unable to find container named "nginx"
```

❌ **Reason**\
The container name is **not** `nginx`.

***

#### Identify the correct container name

```bash
kubectl describe deployment nginx-deployment
```

Relevant section:

```
Containers:
  nginx-container:
    Image:         nginx:1.19
```

✅ Correct container name: **`nginx-container`**

***

#### Apply the correct image update

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:latest
```

```
deployment.apps/nginx-deployment image updated
```

***

#### Monitor rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

```
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

***

### ✅ Final Verification

#### Confirm image update

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

```
Image:         nginx:latest
```

***

#### Confirm replicas

```bash
kubectl get deployment nginx-deployment
```

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           6m36s
```

***

#### Confirm service NodePort

```bash
kubectl get service nginx-service
```

```
NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.96.95.76   <none>        80:32165/TCP   6m43s
```

***

### 🎉 Final Outcome

✔ Service NodePort updated to **32165**\
✔ Deployment scaled to **5 replicas**\
✔ Image updated to **nginx:latest**\
✔ Deployment and Service **not deleted**\
✔ Rolling update completed successfully

***

### 🧠 Key Learnings (Exam & Real-World)

* **Pod names are not Deployment names**
* `kubectl set image` requires the **container name**
* Use `kubectl edit` for live Service updates
* `kubectl scale` is the safest way to adjust replicas
* Always verify with `kubectl describe` and `kubectl rollout status`

***

### 📌 One-Line Exam Tip

> **Service edit → Deployment scale → Image update → Rollout verify**

***

If you want next:

* 📄 Export this as a **GitBook-ready `.md` file**
* 🎓 Add **CKA / CKAD exam traps**
* 📊 Add **architecture diagrams**

Just tell me 👍
