# Rolling Updates And Rolling Back Deployments in Kubernetes

There is a production deployment planned for next week. The Nautilus DevOps team wants to test the deployment update and rollback on Dev environment first so that they can identify the risks in advance. Below you can find more details about the plan they want to execute.

1. Create a namespace `nautilus`. Create a deployment called `httpd-deploy` under this new namespace, It should have one container called `httpd`, use `httpd:2.4.27` image and `4` replicas. The deployment should use `RollingUpdate` strategy with `maxSurge=1`, and `maxUnavailable=2`. Also create a `NodePort` type service named `httpd-service` and expose the deployment on `nodePort: 30008`.
2. Now upgrade the deployment to version `httpd:2.4.43` using a rolling update.
3. Finally, once all pods are updated undo the recent update and roll back to the previous/original version.

`Note:`

a. The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

b. Please make sure you only use the specified image(s) for this deployment and as per the sequence mentioned in the task description. If you mistakenly use a wrong image and fix it later, that will also distort the revision history which can eventually fail this task.



## Kubernetes Deployment Update and Rollback

### Overview

This document covers the end-to-end procedure for deploying an application to a Kubernetes cluster, performing a rolling update, and executing a rollback. The exercise was carried out on the `nautilus` namespace in a Dev environment prior to a production release, following the Nautilus DevOps team's validation plan.

***

### Prerequisites

* `kubectl` configured on the jump host and connected to the target Kubernetes cluster.
* Sufficient RBAC permissions to create namespaces, deployments, and services.

***

### Step 1: Create the Namespace

```bash
kubectl create namespace nautilus
```

**Output:**

```
namespace/nautilus created
```

***

### Step 2: Create the Deployment

Create a deployment named `httpd-deploy` with 4 replicas using the `httpd:2.4.27` image.

```bash
kubectl create deployment httpd-deploy \
  --image=httpd:2.4.27 \
  --replicas=4 \
  -n nautilus
```

**Output:**

```
deployment.apps/httpd-deploy created
```

***

### Step 3: Configure the Rolling Update Strategy

The `kubectl set` command does not support strategy flags directly. Use `kubectl patch` instead to apply `RollingUpdate` with `maxSurge=1` and `maxUnavailable=2`.

```bash
kubectl patch deployment httpd-deploy -n nautilus \
  -p '{"spec":{"strategy":{"type":"RollingUpdate","rollingUpdate":{"maxSurge":1,"maxUnavailable":2}}}}'
```

**Output:**

```
deployment.apps/httpd-deploy patched
```

**Verify the strategy was applied:**

```bash
kubectl get deployment httpd-deploy -n nautilus -o yaml | grep -A 5 strategy
```

**Output:**

```yaml
strategy:
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 2
  type: RollingUpdate
template:
```

***

### Step 4: Create the NodePort Service

The `--node-port` flag is not supported by `kubectl expose`. Apply the service manifest directly using a heredoc.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
  namespace: nautilus
spec:
  type: NodePort
  selector:
    app: httpd-deploy
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
EOF
```

**Output:**

```
service/httpd-service created
```

**Verify the deployment and service:**

```bash
kubectl get deployment httpd-deploy -n nautilus
kubectl get svc httpd-service -n nautilus
```

**Output:**

```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
httpd-deploy   4/4     4            4           2m8s

NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
httpd-service   NodePort   10.43.141.145   <none>        80:30008/TCP   9s
```

***

### Step 5: Perform the Rolling Update

Update the deployment image from `httpd:2.4.27` to `httpd:2.4.43`.

```bash
kubectl set image deployment/httpd-deploy httpd=httpd:2.4.43 -n nautilus
```

**Output:**

```
deployment.apps/httpd-deploy image updated
```

**Wait for the rollout to complete before proceeding:**

```bash
kubectl rollout status deployment/httpd-deploy -n nautilus
```

**Output:**

```
deployment "httpd-deploy" successfully rolled out
```

> **Important:** Always confirm the rollout is complete before initiating a rollback. Running `rollout undo` during an in-progress update will result in incomplete revision history and an unreliable rollback target.

***

### Step 6: Roll Back to the Previous Version

```bash
kubectl rollout undo deployment/httpd-deploy -n nautilus
```

**Output:**

```
deployment.apps/httpd-deploy rolled back
```

**Confirm the rollback completed:**

```bash
kubectl rollout status deployment/httpd-deploy -n nautilus
```

**Output:**

```
deployment "httpd-deploy" successfully rolled out
```

***

### Step 7: Verify Final State

**Check the active image:**

```bash
kubectl describe deployment httpd-deploy -n nautilus | grep Image
```

**Output:**

```
Image:         httpd:2.4.27
```

**Review the revision history:**

```bash
kubectl rollout history deployment/httpd-deploy -n nautilus
```

**Output:**

```
deployment.apps/httpd-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

**Check service endpoints:**

```bash
kubectl get endpoints httpd-service -n nautilus
```

**Output:**

```
NAME            ENDPOINTS                                               AGE
httpd-service   10.22.0.37:80,10.22.0.38:80,10.22.0.39:80 + 1 more...   2m31s
```

***

### Revision History Explained

| Revision | Image        | Description                                                  |
| -------- | ------------ | ------------------------------------------------------------ |
| 1        | httpd:2.4.27 | Initial deployment — absorbed into revision 3 after rollback |
| 2        | httpd:2.4.43 | Rolling update                                               |
| 3        | httpd:2.4.27 | Rollback — current active revision                           |

Revision 1 disappearing from the history is expected Kubernetes behavior. When a rollback restores an identical pod template, Kubernetes re-uses the existing revision entry rather than creating a duplicate, causing the original revision number to be superseded by the new one.

***

### Final State Summary

| Resource        | Detail        |
| --------------- | ------------- |
| Namespace       | nautilus      |
| Deployment      | httpd-deploy  |
| Replicas        | 4             |
| Active Image    | httpd:2.4.27  |
| Update Strategy | RollingUpdate |
| maxSurge        | 1             |
| maxUnavailable  | 2             |
| Service Name    | httpd-service |
| Service Type    | NodePort      |
| NodePort        | 30008         |

***

### Key Lessons

**Use `kubectl patch` for strategy configuration.** The `kubectl set` command does not expose rolling update strategy flags. Patching the deployment spec directly is the correct approach.

**Use a manifest for NodePort services.** The `kubectl expose` command does not support the `--node-port` flag. Applying a YAML manifest via stdin gives full control over service configuration.

**Always wait for rollout completion before rolling back.** Running `kubectl rollout undo` against an in-progress deployment can corrupt the revision history. Use `kubectl rollout status` to confirm completion first.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

