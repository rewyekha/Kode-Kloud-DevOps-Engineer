# Troubleshoot Deployment issues in Kubernetes

Last week, the Nautilus DevOps team deployed a redis app on Kubernetes cluster, which was working fine so far. This morning one of the team members was making some changes in this existing setup, but he made some mistakes and the app went down. We need to fix this as soon as possible. Please take a look.

The deployment name is `redis-deployment`. The pods are not in running state right now, so please look into the issue and fix the same.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

\# Nautilus Kubernetes: Fix Broken Redis Deployment\
**Platform:** KodeKloud **Topic:** Kubernetes, Troubleshooting, Deployment, ConfigMap **Difficulty:** Intermediate **Lab Type:** Troubleshooting \
\---\
\## Table of Contents\
1\. #lab-overview2. Lab Objectives3. #prerequisites4. #problem-statement5. Troubleshooting Process6. #root-cause-analysis7. #resolution-steps8. #verification9. #lab-completion-status10. #key-learnings11. #common-mistakes--prevention\
\---\
\## Lab Overview\
The Nautilus DevOps team maintains a Redis application deployed on a Kubernetes cluster. Recent configuration changes introduced errors that caused the Redis deployment to break. As a result, the Redis pod remained stuck in a non‑running state.\
The goal of this lab was to identify the misconfiguration, correct the deployment, and restore the Redis pod to a healthy **Running** state.\
\---\
\## Lab Objectives\
\- Diagnose why the Redis pod is not starting- Identify configuration errors in the Deployment specification- Fix the Deployment so the pod becomes **Running (1/1 Ready)**- Verify that the Redis application is restored\
\---\
\## Prerequisites\
\- Access to the jump host with `kubectl` configured- A functioning Kubernetes cluster- Basic knowledge of Kubernetes deployments and ConfigMaps\
\---\
\## Problem Statement\
\### Deployment Status\
\`\`\`bashkubectl get deployment redis-deployment

**Output**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   0/1     1            0           3m
```

#### Pod Status

kubectl get pods

**Output**

```
NAME                                      READY   STATUS             RESTARTS   AGE
redis-deployment-795ffcb56c-jjsrb         0/1     ContainerCreating   0          3m
```

The Redis pod failed to reach a running state and was stuck during container creation.

***

### Troubleshooting Process

#### Step 1: Describe the Pod

To identify the failure reason, the pod was described to inspect events and errors.

kubectl describe pod $(kubectl get pods -l app=redis -o jsonpath="{.items\[0].metadata.name}")

**Relevant Event**

```
Warning  FailedMount  ...  MountVolume.SetUp failed for volume "config" :
configmap "redis-cofig" not found
```

***

#### Step 2: Inspect the Deployment Configuration

kubectl get deployment redis-deployment -o yaml

Two configuration issues were identified in the deployment:

1. Incorrect container image name
   * `redis:alpin` (invalid)
2. Incorrect ConfigMap reference
   * `redis-cofig` (non‑existent resource)

***

### Root Cause Analysis

The Redis deployment failed due to configuration typos.

| Configuration Area | Incorrect Value | Correct Value  | Impact               |
| ------------------ | --------------- | -------------- | -------------------- |
| Container Image    | `redis:alpin`   | `redis:alpine` | Image pull error     |
| ConfigMap Name     | `redis-cofig`   | `redis-config` | Volume mount failure |

The **primary blocking issue** was the incorrect ConfigMap name, which caused the pod to fail during volume mounting (`FailedMount`).

***

### Resolution Steps

#### Step 1: Edit the Deployment

kubectl edit deployment redis-deployment

#### Step 2: Apply Configuration Fixes

**1. Fix the Redis image**

\# Beforeimage: redis:alpin\
\# Afterimage: redis:alpine

**2. Fix the ConfigMap reference**

\# Beforename: redis-cofig\
\# Aftername: redis-config

The changes were saved and exited using `:wq`.

Kubernetes automatically triggered a rollout after the deployment specification was updated.

***

### Verification

#### Pod Status Check

kubectl get pods

**Output**

```
NAME                                      READY   STATUS    RESTARTS   AGE
redis-deployment-5476b4ddd6-45qb6         1/1     Running   0          2m
```

***

#### Pod Description Validation

kubectl describe pod redis-deployment-5476b4ddd6-45qb6

**Confirmed Results**

* Pod status: **Running**
* Container image: `redis:alpine`
* ConfigMap `redis-config` mounted successfully
* No `FailedMount` or error events present

***

#### Deployment Status

kubectl get deployment redis-deployment

**Output**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           10m
```

***

### Lab Completion Status

| Requirement                 | Status |
| --------------------------- | ------ |
| Pod running successfully    | ✅      |
| Deployment ready (1/1)      | ✅      |
| Correct Redis image         | ✅      |
| Correct ConfigMap reference | ✅      |

The Redis application has been fully restored.

***

### Key Learnings

* `kubectl describe pod` is the most effective command for identifying pod startup issues.
* Typos in ConfigMap, Secret, or volume references commonly result in `FailedMount` errors.
* `kubectl edit` allows quick fixes for small deployment issues.
* Any update to a Deployment spec automatically triggers a rollout.
* Image tag validation is essential to avoid runtime failures.

***

### Common Mistakes & Prevention

* **Incorrect resource names**\
  Always ensure ConfigMaps, Secrets, and PVC names match exactly.
* **Invalid image tags**\
  Verify container images exist in the registry.
* **Ignoring pod events**\
  Always inspect events when pods are not running.
* **Not saving deployment edits**\
  Confirm changes are saved when exiting `kubectl edit`.

**Tip:** Use the following command to monitor deployment progress:

kubectl rollout status deployment/redis-deployment

***

**Lab Completed On:** April 10, 2026\
**Platform:** KodeKloud Kubernetes\
**Author:** Reyas Khan



<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
