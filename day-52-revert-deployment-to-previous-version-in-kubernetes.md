# Day 52: Revert Deployment to Previous Version in Kubernetes

Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.

There exists a deployment named `nginx-deployment`; initiate a rollback to the previous revision.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



## Kubernetes Deployment Rollback – `nginx-deployment`

### Problem Statement

Earlier today, the Nautilus DevOps team deployed a **new release** of an application. Shortly after deployment, a customer reported a **bug in the latest version**.

To resolve the issue quickly, the team decided to **rollback the deployment to the previous stable revision**.

#### Objective

Rollback the Kubernetes deployment:

```
nginx-deployment
```

to the **previous revision**.

> The `kubectl` CLI on the **jump-host** is already configured to communicate with the Kubernetes cluster.

***

## Step 1: Check Existing Kubernetes Resources

First, verify the current running resources.

```bash
kubectl get all
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7795857fdb-dbjnj   1/1     Running   0          3m25s
pod/nginx-deployment-7795857fdb-fwzcq   1/1     Running   0          3m22s
pod/nginx-deployment-7795857fdb-xxqn5   1/1     Running   0          3m21s

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.43.0.1      <none>        443/TCP        70m
service/nginx-service   NodePort    10.43.55.107   <none>        80:30008/TCP   3m35s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           3m35s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-7795857fdb   3         3         3       3m25s
replicaset.apps/nginx-deployment-fc677cbc9    0         0         0       3m35s
```

#### Explanation

* **Pods** → 3 running pods created by the deployment.
* **Service** → `nginx-service` exposes the application using NodePort.
* **Deployment** → `nginx-deployment` managing 3 replicas.
* **ReplicaSets**
  * `nginx-deployment-7795857fdb` → Current version
  * `nginx-deployment-fc677cbc9` → Previous version (scaled to 0)

***

## Step 2: Inspect Deployment Details

Next, check the deployment configuration.

```bash
kubectl describe deploy nginx-deployment
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl describe deploy nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 17 Mar 2026 00:20:13 +0000
Labels:                 app=nginx-app
                        type=front-end
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --record=true
Selector:               app=nginx-app
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate

Containers:
 nginx-container:
  Image: nginx:alpine
```

#### Explanation

Key details:

* **Current Revision:** `2`
* **Image Used:** `nginx:alpine`
* **Change Cause:** Image updated using:

```
kubectl set image deployment nginx-deployment nginx-container=nginx:alpine
```

This confirms that the **latest release introduced the buggy image**.

***

## Step 3: Check Deployment Revision History

Before rolling back, view the revision history.

```bash
kubectl rollout history deployment nginx-deployment
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl rollout history deployment nginx-deployment

deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --record=true
```

#### Explanation

Two revisions exist:

| Revision | Description                          |
| -------- | ------------------------------------ |
| 1        | Original stable version              |
| 2        | Updated version using `nginx:alpine` |

Since **Revision 2 contains the bug**, we rollback to the **previous revision**.

***

## Step 4: Perform Deployment Rollback

Run the rollback command:

```bash
kubectl rollout undo deployment nginx-deployment
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl rollout undo deployment nginx-deployment
deployment.apps/nginx-deployment rolled back
```

#### Explanation

Kubernetes automatically:

1. Restores the previous deployment configuration
2. Scales up the previous ReplicaSet
3. Scales down the faulty ReplicaSet

***

## Step 5: Verify Rollout Status

Confirm the rollback completed successfully.

```bash
kubectl rollout status deployment nginx-deployment
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl rollout status deployment nginx-deployment
deployment "nginx-deployment" successfully rolled out
```

#### Explanation

This confirms the deployment rollout finished successfully.

***

## Step 6: Verify Updated Revision History

Check the revision history again.

```bash
kubectl rollout history deployment nginx-deployment
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl rollout history deployment nginx-deployment

deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --record=true
3         <none>
```

#### Explanation

After rollback:

* Kubernetes **creates a new revision**
* The rollback becomes **Revision 3**

Important concept:

> Kubernetes does **not reuse old revisions** during rollback.\
> Instead, it creates a **new revision representing the rollback state**.

***

## Step 7: Verify Application Image

Check which container image is currently running.

```bash
kubectl describe deployment nginx-deployment | grep Image
```

#### Terminal Output

```bash
thor@jump-host ~$ kubectl describe deployment nginx-deployment | grep Image
    Image:         nginx:1.16
```

#### Explanation

The deployment has successfully reverted from:

```
nginx:alpine
```

to

```
nginx:1.16
```

This confirms the rollback restored the **previous stable version**.

***

## How Kubernetes Performs a Rollback

When a rollback occurs, Kubernetes performs the following steps:

1. Identifies the **previous ReplicaSet**
2. Scales it **back up**
3. Scales the **current ReplicaSet down**
4. Creates a **new revision entry**
5. Updates the deployment status

***

## Final Result

| Component   | Status                   |
| ----------- | ------------------------ |
| Deployment  | Rolled back successfully |
| Pods        | Running                  |
| Image       | nginx:1.16               |
| Revision    | 3                        |
| Application | Stable version restored  |

***

## Final Command Used

```bash
kubectl rollout undo deployment nginx-deployment
```

***

## Key Kubernetes Commands

| Command                       | Purpose                    |
| ----------------------------- | -------------------------- |
| `kubectl get all`             | View all resources         |
| `kubectl describe deployment` | Inspect deployment details |
| `kubectl rollout history`     | View deployment revisions  |
| `kubectl rollout undo`        | Rollback deployment        |
| `kubectl rollout status`      | Check rollout progress     |

***

## Key Takeaways

* Kubernetes **tracks deployment revisions automatically**
* `kubectl rollout undo` quickly restores a previous version
* Rollbacks **create new revisions rather than reusing old ones**
* Always verify using `kubectl rollout status`

***

✔ Deployment successfully rolled back to the **previous stable version**.

<figure><img src=".gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

