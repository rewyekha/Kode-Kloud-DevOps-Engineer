# Troubleshoot Deployment issues in Kubernetes

Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.

The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix the same.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



````markdown
# Nautilus Kubernetes: Fix Broken Redis Deployment

**Platform:** KodeKloud | **Topic:** Kubernetes, Troubleshooting, Deployment, ConfigMap  
**Difficulty:** Intermediate | **Lab Type:** Troubleshooting  

## Table of Contents

1. [Lab Overview](#lab-overview)
2. [Lab Objectives](#lab-objectives)
3. [Prerequisites](#prerequisites)
4. [Initial Issue](#initial-issue)
5. [Troubleshooting Steps](#troubleshooting-steps)
6. [Root Cause Analysis](#root-cause-analysis)
7. [Solution Steps](#solution-steps)
8. [Verification](#verification)
9. [Lab Complete](#lab-complete)
10. [Key Learnings](#key-learnings)
11. [Common Mistakes & Prevention](#common-mistakes--prevention)

## Lab Overview

The Nautilus DevOps team had a working Redis deployment on the Kubernetes cluster. A team member made changes that broke the application. The `redis-deployment` was stuck with pods in **Pending/ContainerCreating** state. The task was to identify the issue and restore the Redis pod to a healthy **Running** state.

## Lab Objectives

- Diagnose why the Redis pod is not running
- Identify configuration errors in the Deployment
- Fix the Deployment so the pod reaches **Running** state (1/1 Ready)
- Ensure the Redis application is back online

## Prerequisites

- Access to the jump-host with `kubectl` pre-configured
- Working Kubernetes cluster (default namespace)

## Initial Issue

```bash
kubectl get deployment redis-deployment
````

**Output:**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           3m
```

```bash
kubectl get pods
```

**Output:**

```
NAME                                      READY   STATUS             RESTARTS   AGE
redis-deployment-795ffcb56c-jjsrb         0/1     ContainerCreating   0          3m
```

### Troubleshooting Steps

#### Step 1: Describe the Pod to see detailed errors

```bash
kubectl describe pod $(kubectl get pods -l app=redis -o jsonpath="{.items[0].metadata.name}")
```

**Key Error Found in Events:**

```
Warning  FailedMount  ...  MountVolume.SetUp failed for volume "config" : 
configmap "redis-cofig" not found
```

#### Step 2: Check the Deployment YAML

```bash
kubectl get deployment redis-deployment -o yaml
```

We observed two critical issues in the spec:

1. Wrong image: `redis:alpin` (should be `redis:alpine`)
2. Wrong ConfigMap name: `redis-cofig` (should be `redis-config`)

### Root Cause Analysis

The deployment had two configuration mistakes:

| Issue               | Wrong Value   | Correct Value  | Impact                                               |
| ------------------- | ------------- | -------------- | ---------------------------------------------------- |
| Redis Image         | `redis:alpin` | `redis:alpine` | Image pull would eventually fail                     |
| ConfigMap Reference | `redis-cofig` | `redis-config` | Volume mount failed → Pod stuck in ContainerCreating |

The main blocker was the **typo in ConfigMap name**, causing `FailedMount`.

### Solution Steps

#### Step 1: Edit the Deployment

```bash
kubectl edit deployment redis-deployment
```

**Changes Made Inside the Editor:**

1.  **Corrected the image:**

    ```yaml
    # From
    image: redis:alpin
    # To
    image: redis:alpine
    ```
2.  **Corrected the ConfigMap name:**

    ```yaml
    # From
    name: redis-cofig
    # To
    name: redis-config
    ```

After editing, saved and exited using `:wq`

#### Step 2: Kubernetes automatically rolled out the changes

Kubernetes detected the change and created a new ReplicaSet + new Pod.

### Verification

#### Check Pod Status

```bash
kubectl get pods
```

**Final Output:**

```
NAME                                      READY   STATUS    RESTARTS   AGE
redis-deployment-5476b4ddd6-45qb6         1/1     Running   0          2m
```

#### Detailed Pod Description

```bash
kubectl describe pod redis-deployment-5476b4ddd6-45qb6
```

**Key Confirmations:**

* Status: **Running**
* Ready: **True**
* Container Image: `redis:alpine`
* Volume "config" mounted from ConfigMap `redis-config`
* No FailedMount errors

#### Deployment Status

```bash
kubectl get deployment redis-deployment
```

**Output:**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           10m
```

### Lab Complete

**Requirement Status**

| Requirement                    | Status |
| ------------------------------ | ------ |
| Pod in Running state           | ✅ Done |
| Deployment ready (1/1)         | ✅ Done |
| Correct image (`redis:alpine`) | ✅ Done |
| Correct ConfigMap reference    | ✅ Done |

The Redis application is now restored and running successfully.

### Key Learnings

* Always check `kubectl describe pod` when a pod is not Running — it shows the real root cause.
* Typo in resource names (ConfigMap, Secret, PVC, etc.) is a very common cause of `FailedMount`.
* `kubectl edit` is a quick way to fix small configuration issues in Deployments.
* Kubernetes automatically rolls out changes when the Deployment spec is updated.
* Image name typos (`alpin` vs `alpine`) can also cause failures later.

### Common Mistakes & Prevention

* **Typo in ConfigMap/Secret name** → Always double-check resource references.
* Using wrong image tags → Verify image exists on Docker Hub or registry.
* Not checking Events in `describe pod` → This is the fastest way to troubleshoot.
* Forgetting to save `:wq` after `kubectl edit`.

**Pro Tip:** After editing a deployment, use `kubectl rollout status deployment/<name>` to monitor the rollout.

***

**Lab Completed on:** April 10, 2026\
**Platform:** KodeKloud Kubernetes Lab<br>

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
