# Day 59: Troubleshoot Deployment Issues in Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.

The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix the same.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

## Kubernetes Troubleshooting: Fix Redis Deployment

> **Platform:** KodeKloud | **Series:** Nautilus DevOps **Difficulty:** Beginner | **Topic:** Kubernetes, Deployments, Troubleshooting, ConfigMaps

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Investigation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#investigation)
   * [Step 1: Check Deployment Status](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-check-deployment-status)
   * [Step 2: Check Pod Status](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-check-pod-status)
   * [Step 3: Describe the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-describe-the-deployment)
   * [Step 4: Verify ConfigMaps](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-configmaps)
4. [Root Cause Analysis](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#root-cause-analysis)
5. [Fix](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#fix)
   * [Step 5: Edit the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-edit-the-deployment)
   * [Step 6: Verify the Fix](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-verify-the-fix)
   * [Step 7: Confirm Pod is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-confirm-pod-is-running)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

Last week, the Nautilus DevOps team deployed a Redis app on a Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but made some mistakes and the app went down. We need to fix this as soon as possible.

* The deployment name is `redis-deployment`
* The pods are **not** in running state
* Investigate the issue and fix it

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

| Component       | Detail                      |
| --------------- | --------------------------- |
| Access Host     | `jump-host`                 |
| User            | `thor`                      |
| Deployment Name | `redis-deployment`          |
| Namespace       | `default`                   |
| kubectl         | Pre-configured on jump-host |

***

### Investigation

#### Step 1: Check Deployment Status

Start by checking the overall state of the deployment:

```bash
kubectl get deployment redis-deployment
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get deployment redis-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           2m3s
```

> `READY: 0/1` and `AVAILABLE: 0` — the deployment exists but no pods are healthy.

***

#### Step 2: Check Pod Status

```bash
kubectl get pods
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
redis-deployment-795ffcb56c-d8b2m   0/1     ContainerCreating   0          3m39s
```

> The pod is stuck in `ContainerCreating` — it has been trying to start for over 3 minutes without success. This is abnormal and indicates a configuration problem.

***

#### Step 3: Describe the Deployment

The `describe` command gives a detailed view of the deployment configuration and any error events:

```bash
kubectl describe deployment redis-deployment
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl describe deployment redis-deployment
Name:                   redis-deployment
Namespace:              default
CreationTimestamp:      Tue, 24 Mar 2026 05:05:05 +0000
Labels:                 app=redis
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=redis
Replicas:               1 desired | 1 updated | 1 total | 0 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=redis
  Containers:
   redis-container:
    Image:      redis:alpin
    Port:       6379/TCP
    Host Port:  0/TCP
    Requests:
      cpu:        300m
    Environment:  <none>
    Mounts:
      /redis-master from config (rw)
      /redis-master-data from data (rw)
  Volumes:
   data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
   config:
    Type:          ConfigMap (a volume populated by a ConfigMap)
    Name:          redis-cofig
    Optional:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   redis-deployment-795ffcb56c (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m47s  deployment-controller  Scaled up replica set redis-deployment-795ffcb56c from 0 to 1
```

> **Two issues identified:**
>
> 1. `Image: redis:alpin` — typo in the image tag (missing `e`)
> 2. `Name: redis-cofig` — typo in the ConfigMap name (missing `n`)

***

#### Step 4: Verify ConfigMaps

Confirm what ConfigMaps actually exist in the cluster:

```bash
kubectl get configmaps
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      38m
redis-config       2      5m4s
```

> The correct ConfigMap name is `redis-config` (with `n`) — confirming the deployment was referencing a non-existent ConfigMap named `redis-cofig`.

***

### Root Cause Analysis

Two typos were introduced into the deployment configuration by the team member:

| # | Field            | Incorrect Value | Correct Value  | Impact                                                                         |
| - | ---------------- | --------------- | -------------- | ------------------------------------------------------------------------------ |
| 1 | `image`          | `redis:alpin`   | `redis:alpine` | Pod stuck in `ContainerCreating` — Docker cannot pull a non-existent image tag |
| 2 | ConfigMap `name` | `redis-cofig`   | `redis-config` | Volume mount fails — Kubernetes cannot find the referenced ConfigMap           |

Both errors prevented the pod from ever reaching `Running` state.

***

### Fix

#### Step 5: Edit the Deployment

Use `kubectl edit` to open the deployment manifest in a `vi` editor and fix both typos:

```bash
kubectl edit deployment redis-deployment
```

Inside the editor, locate and correct the two lines:

```yaml
# Fix 1 — Image tag typo
# Before:
image: redis:alpin
# After:
image: redis:alpine

# Fix 2 — ConfigMap name typo
# Before:
name: redis-cofig
# After:
name: redis-config
```

Save and exit: press `Esc`, then type `:wq` and hit `Enter`.

**Terminal Output:**

```
thor@jump-host ~$ kubectl edit deployment redis-deployment
deployment.apps/redis-deployment edited
```

***

#### Step 6: Verify the Fix

Describe the deployment again to confirm both values are now correct:

```bash
kubectl describe deployment redis-deployment
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl describe deployment redis-deployment
Name:                   redis-deployment
Namespace:              default
CreationTimestamp:      Tue, 24 Mar 2026 05:05:05 +0000
Labels:                 app=redis
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=redis
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=redis
  Containers:
   redis-container:
    Image:      redis:alpine
    Port:       6379/TCP
    Host Port:  0/TCP
    Requests:
      cpu:        300m
    Environment:  <none>
    Mounts:
      /redis-master from config (rw)
      /redis-master-data from data (rw)
  Volumes:
   data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
   config:
    Type:          ConfigMap (a volume populated by a ConfigMap)
    Name:          redis-config
    Optional:      false
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  redis-deployment-795ffcb56c (0/0 replicas created)
NewReplicaSet:   redis-deployment-5476b4ddd6 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m30s  deployment-controller  Scaled up replica set redis-deployment-795ffcb56c from 0 to 1
  Normal  ScalingReplicaSet  13s    deployment-controller  Scaled up replica set redis-deployment-5476b4ddd6 from 0 to 1
  Normal  ScalingReplicaSet  9s     deployment-controller  Scaled down replica set redis-deployment-795ffcb56c from 1 to 0
```

**Confirmation from describe output:**

| Field                  | Before Fix    | After Fix      |
| ---------------------- | ------------- | -------------- |
| `revision`             | `1`           | `2`            |
| `Image`                | `redis:alpin` | `redis:alpine` |
| ConfigMap `Name`       | `redis-cofig` | `redis-config` |
| `Available` condition  | `False`       | `True`         |
| `available` replicas   | `0`           | `1`            |
| `unavailable` replicas | `1`           | `0`            |

The events section also shows a clean rolling update — the old broken ReplicaSet was scaled down and a new healthy one was scaled up.

***

#### Step 7: Confirm Pod is Running

Watch the pod status in real time:

```bash
kubectl get pods -w
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pods -w
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-5476b4ddd6-wnxlc   1/1     Running   0          27s
^C
thor@jump-host ~$
```

> `STATUS: Running` and `READY: 1/1` — the Redis pod is fully healthy.

***

### Lab Complete

| Task                                        | Result |
| ------------------------------------------- | ------ |
| Identified pod stuck in `ContainerCreating` |        |
| Found typo in image tag: `redis:alpin`      |        |
| Found typo in ConfigMap name: `redis-cofig` |        |
| Fixed both issues via `kubectl edit`        |        |
| Pod now in `Running` state `1/1`            |        |

***

### Key Concepts

#### Why `ContainerCreating` Hangs

A pod stays in `ContainerCreating` when Kubernetes is unable to fully initialize the container. Common causes include:

| Cause                       | Symptom                         |
| --------------------------- | ------------------------------- |
| Invalid image tag           | Cannot pull image from registry |
| Missing ConfigMap or Secret | Volume mount fails silently     |
| Missing PersistentVolume    | Storage cannot be attached      |
| Network plugin issues       | CNI not ready                   |

In this lab, **both** an invalid image tag and a missing ConfigMap reference caused the pod to hang indefinitely.

#### `kubectl describe` — Your First Troubleshooting Tool

`kubectl describe` is the most useful first command when a pod or deployment is unhealthy. Key sections to check:

```
Containers:
  Image:        ← check for typos in image name/tag
  Mounts:       ← check volume mount paths

Volumes:
  ConfigMap:
    Name:       ← verify ConfigMap name matches kubectl get configmaps

Conditions:
  Available:    ← False means no healthy replicas

Events:         ← shows what Kubernetes has attempted and any errors
```

#### Rolling Update Behaviour

When `kubectl edit` saves changes, Kubernetes automatically performs a **rolling update**:

```
Old ReplicaSet (broken):   1 → 0  (scaled down)
New ReplicaSet (fixed):    0 → 1  (scaled up)
```

This is visible in the Events section of the describe output — no manual restart was needed.

#### ConfigMap Volume Mounts

A ConfigMap referenced as a volume **must exist** in the same namespace before the pod can start. If the name is wrong or the ConfigMap is missing, the pod will remain in `ContainerCreating` with no obvious error in basic `get pods` output — making `describe` essential for diagnosis.

```yaml
volumes:
  - name: config
    configMap:
      name: redis-config   # must exactly match an existing ConfigMap name
      optional: false      # false = pod will NOT start if ConfigMap is missing
```

***

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

***

[_Reyas Khan_](https://reyaskhan.me) _|_ [_GitHub_](https://github.com/rewyekha)
