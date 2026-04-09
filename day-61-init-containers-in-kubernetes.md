# Day 61: Init Containers in Kubernetes

There are some applications that need to be deployed on Kubernetes cluster and these apps have some pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images so the DevOps team has come up with a solution to use init containers to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.

1. Create a `Deployment` named as `ic-deploy-devops`.
2. Configure `spec` as replicas should be `1`, labels `app` should be `ic-devops`, template's metadata lables `app` should be the same `ic-devops`.
3. The `initContainers` should be named as `ic-msg-devops`, use image `ubuntu` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'echo Init Done - Welcome to xFusionCorp Industries > /ic/media'`. The volume mount should be named as `ic-volume-devops` and mount path should be `/ic`.
4. Main container should be named as `ic-main-devops`, use image `ubuntu` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'while true; do cat /ic/media; sleep 5; done'`. The volume mount should be named as `ic-volume-devops` and mount path should be `/ic`.
5. Volume to be named as `ic-volume-devops` and it should be an emptyDir type.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



## Kubernetes: Deploy Application with Init Containers

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Kubernetes, Deployments, Init Containers, emptyDir Volumes

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Write the Deployment Manifest](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-write-the-deployment-manifest)
   * [Step 2: Apply the Manifest](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-apply-the-manifest)
   * [Step 3: Verify the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-verify-the-deployment)
   * [Step 4: Verify the Pod is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-the-pod-is-running)
   * [Step 5: Check Main Container Logs](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-check-main-container-logs)
   * [Step 6: Review the Final Manifest](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-review-the-final-manifest)
   * [Step 7: Final State Confirmation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-final-state-confirmation)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

There are some applications that need to be deployed on a Kubernetes cluster. These apps have pre-requisites where some configurations need to be changed before deploying the app container. Some of these changes cannot be made inside the images, so the DevOps team has come up with a solution to use **init containers** to perform these tasks during deployment. Below is a sample scenario that the team is going to test first.

**Requirements:**

1. Create a `Deployment` named `ic-deploy-devops`.
2. Configure `spec` as: replicas should be `1`, labels `app` should be `ic-devops`, template's metadata labels `app` should be the same `ic-devops`.
3. The `initContainers` should be named `ic-msg-devops`, use image `ubuntu` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'echo Init Done - Welcome to xFusionCorp Industries > /ic/media'`. The volume mount should be named `ic-volume-devops` and mount path should be `/ic`.
4. The main container should be named `ic-main-devops`, use image `ubuntu` with `latest` tag and use command `'/bin/bash'`, `'-c'` and `'while true; do cat /ic/media; sleep 5; done'`. The volume mount should be named `ic-volume-devops` and mount path should be `/ic`.
5. Volume to be named `ic-volume-devops` and it should be an `emptyDir` type.

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

| Server Name          | Hostname    | User      | Password     | Purpose                               |
| -------------------- | ----------- | --------- | ------------ | ------------------------------------- |
| Application Server 1 | `stapp01`   | `tony`    | `Ir0nM@n`    | Hosts Nautilus Application 1          |
| Application Server 2 | `stapp02`   | `steve`   | `Am3ric@`    | Hosts Nautilus Application 2          |
| Application Server 3 | `stapp03`   | `banner`  | `BigGr33n`   | Hosts Nautilus Application 3          |
| LoadBalancer Server  | `stlb01`    | `loki`    | `Mischi3f`   | Distributes traffic for Nautilus HTTP |
| Database Server      | `stdb01`    | `peter`   | `Sp!dy`      | Hosts Nautilus Database               |
| Storage Server       | `ststor01`  | `natasha` | `Bl@kW`      | Stores data for Nautilus Servers      |
| Backup Server        | `stbkp01`   | `clint`   | `H@wk3y3`    | Manages backups for Nautilus Servers  |
| Mail Server          | `stmail01`  | `groot`   | `Gr00T123`   | Manages email services                |
| Jump Host            | `jump-host` | `thor`    | `mjolnir123` | Provides secure access to Stork DC    |
| Jenkins Server       | `jenkins`   | `jenkins` | `j@rv!s`     | Runs Jenkins for CI/CD pipeline       |

> **Target:** All `kubectl` commands are executed from `jump-host`, which is pre-configured to communicate with the Kubernetes cluster.

***

### Solution

#### Step 1: Write the Deployment Manifest

Create the deployment manifest file `ic-deploy-devops.yaml` using `vim`. The manifest defines the deployment, init container, main container, and shared emptyDir volume in a single file:

```bash
vim ic-deploy-devops.yaml
```

**Manifest content:**

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-devops
  template:
    metadata:
      labels:
        app: ic-devops
    spec:
      initContainers:
      - name: ic-msg-devops
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/media"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      containers:
      - name: ic-main-devops
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/media; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      volumes:
      - name: ic-volume-devops
        emptyDir: {}
```

**Terminal Output:**

```bash
thor@jump-host ~$ vim ic-deploy-devops.yaml
thor@jump-host ~$
```

***

#### Step 2: Apply the Manifest

Apply the manifest to create the deployment in the Kubernetes cluster:

```bash
kubectl apply -f ic-deploy-devops.yaml
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl apply -f ic-deploy-devops.yaml
deployment.apps/ic-deploy-devops created
```

Kubernetes accepted the manifest and created the deployment `ic-deploy-devops`.

***

#### Step 3: Verify the Deployment

Check the deployment status to confirm it is ready with the correct replica count:

```bash
kubectl get deployments
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-devops   1/1     1            1           14s
```

The deployment shows `READY: 1/1`, `UP-TO-DATE: 1`, and `AVAILABLE: 1` at 14 seconds — confirming the init container ran to completion and the main container started successfully.

***

#### Step 4: Verify the Pod is Running

List the pod created by the deployment using its label selector:

```bash
kubectl get pods -l app=ic-devops
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get pods -l app=ic-devops
NAME                                READY   STATUS    RESTARTS   AGE
ic-deploy-devops-55f857555f-622lh   1/1     Running   0          26s
```

The pod `ic-deploy-devops-55f857555f-622lh` is in `Running` state with `1/1` containers ready and `0` restarts. The pod name follows the pattern `<deployment-name>-<replicaset-hash>-<pod-hash>`.

***

#### Step 5: Check Main Container Logs

Verify that the init container successfully wrote the message to the shared volume, and that the main container is reading and printing it in a continuous loop. Use the pod name from the previous step:

```bash
kubectl logs ic-deploy-devops-55f857555f-622lh -c ic-main-devops
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl logs ic-deploy-devops-55f857555f-622lh -c ic-main-devops
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
```

The main container is printing `Init Done - Welcome to xFusionCorp Industries` every 5 seconds — confirming the full workflow:

1. The init container `ic-msg-devops` wrote the message to `/ic/media` on the shared volume.
2. The init container completed and exited.
3. Kubernetes started the main container `ic-main-devops`.
4. The main container is reading `/ic/media` every 5 seconds and printing its contents to stdout.

> Note: An initial attempt was made using `kubectl logs <pod-name> -c ic-main-devops` literally, which failed with `bash: pod-name: No such file or directory`. The placeholder was then replaced with the actual pod name for the correct command.

***

#### Step 6: Review the Final Manifest

Inspect the applied manifest to confirm its contents:

```bash
cat ic-deploy-devops.yaml
```

**Terminal Output:**

```yml
thor@jump-host ~$ cat ic-deploy-devops.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-devops
  template:
    metadata:
      labels:
        app: ic-devops
    spec:
      initContainers:
      - name: ic-msg-devops
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "echo Init Done - Welcome to xFusionCorp Industries > /ic/media"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      containers:
      - name: ic-main-devops
        image: ubuntu:latest
        command: ["/bin/bash", "-c", "while true; do cat /ic/media; sleep 5; done"]
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
      volumes:
      - name: ic-volume-devops
        emptyDir: {}
```

***

#### Step 7: Final State Confirmation

Run final checks on both the pod and deployment to confirm everything is stable after the full runtime:

```bash
kubectl get pods -l app=ic-devops
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get pods -l app=ic-devops
NAME                                READY   STATUS    RESTARTS   AGE
ic-deploy-devops-55f857555f-622lh   1/1     Running   0          99s
```

```bash
kubectl get deployments
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
ic-deploy-devops   1/1     1            1           106s
thor@jump-host ~$
```

At 106 seconds, the deployment remains `1/1 Ready` with `0` restarts — confirming the init container pattern is working correctly and the main container is running stably.

***

### Lab Complete

| Requirement            | Detail                                                    | Status    |
| ---------------------- | --------------------------------------------------------- | --------- |
| Deployment name        | `ic-deploy-devops`                                        | Confirmed |
| Replicas               | `1`                                                       | Confirmed |
| Selector label         | `app: ic-devops`                                          | Confirmed |
| Template label         | `app: ic-devops`                                          | Confirmed |
| Init container name    | `ic-msg-devops`                                           | Confirmed |
| Init container image   | `ubuntu:latest`                                           | Confirmed |
| Init container command | `echo Init Done... > /ic/media`                           | Confirmed |
| Init volume mount name | `ic-volume-devops` at `/ic`                               | Confirmed |
| Main container name    | `ic-main-devops`                                          | Confirmed |
| Main container image   | `ubuntu:latest`                                           | Confirmed |
| Main container command | `while true; do cat /ic/media; sleep 5; done`             | Confirmed |
| Main volume mount name | `ic-volume-devops` at `/ic`                               | Confirmed |
| Volume name            | `ic-volume-devops`                                        | Confirmed |
| Volume type            | `emptyDir: {}`                                            | Confirmed |
| Pod status             | `1/1 Running`, `0 restarts`                               | Confirmed |
| Log output             | `Init Done - Welcome to xFusionCorp Industries` repeating | Confirmed |

***

### Key Concepts

#### What are Init Containers?

Init containers are specialised containers that run and complete **before** the main application containers start in a pod. They are defined under `initContainers` in the pod spec and are executed sequentially — each init container must exit with a success code (exit 0) before the next one starts, and before any regular containers are started.

```bash
Pod Lifecycle with Init Container:

[Init Container: ic-msg-devops]
    Runs command → writes to /ic/media → exits 0
            |
            | only starts after init container exits successfully
            v
[Main Container: ic-main-devops]
    Reads /ic/media every 5 seconds → prints to stdout → runs forever
```

#### Why emptyDir Was Used

`emptyDir` is an ephemeral volume that is created when a pod is assigned to a node and exists for the lifetime of that pod. It starts empty — hence the name. Any container within the same pod can read and write to it.

In this lab it serves as the communication channel between the init container and the main container:

```bash
Init Container                emptyDir Volume              Main Container
ic-msg-devops                 ic-volume-devops             ic-main-devops
    |                              |                             |
    | mounts at /ic                |        mounts at /ic        |
    |──────────────────────────────▶                            |
    | writes "Init Done..." to /ic/media                         |
    |                              |                             |
    | exits 0                      |    reads /ic/media          |
                                   |◀────────────────────────────|
                                        every 5 seconds
```

| Volume Type             | Lifetime      | Shared Across          | Use Case                               |
| ----------------------- | ------------- | ---------------------- | -------------------------------------- |
| `emptyDir`              | Pod lifetime  | Containers in same pod | Temporary inter-container data sharing |
| `hostPath`              | Node lifetime | Pods on same node      | Access node filesystem                 |
| `persistentVolumeClaim` | Independent   | Pods across nodes      | Persistent application data            |

#### Init Container vs Sidecar Container

| Feature       | Init Container                 | Sidecar Container             |
| ------------- | ------------------------------ | ----------------------------- |
| Defined under | `initContainers`               | `containers`                  |
| Runs          | Before main containers         | Alongside main containers     |
| Lifecycle     | Runs once then exits           | Runs for pod lifetime         |
| Purpose       | Setup, pre-configuration       | Logging, proxying, monitoring |
| Must succeed  | Yes — pod fails if it does not | No — independent lifecycle    |

#### The `-c` Flag in `kubectl logs`

When a pod has more than one container — including init containers — you must specify which container's logs to retrieve using the `-c` flag:

```bash
kubectl logs <pod-name> -c <container-name>
```

Without `-c`, `kubectl logs` defaults to the first container defined under `containers`. Init container logs can also be retrieved:

```bash
# Get init container logs
kubectl logs ic-deploy-devops-55f857555f-622lh -c ic-msg-devops

# Get main container logs
kubectl logs ic-deploy-devops-55f857555f-622lh -c ic-main-devops
```

#### Label Selectors in kubectl

The `-l` flag in `kubectl get pods` filters results by label:

```bash
kubectl get pods -l app=ic-devops
```

This returns only pods that have the label `app=ic-devops` — which matches the label defined in the pod template of the deployment. Label selectors are how Kubernetes connects deployments, replica sets, services, and other resources to their target pods.

***

_Lab completed on 2026-03-26 | Cluster: Kubernetes | Namespace: default | Pod: ic-deploy-devops-55f857555f-622lh_

<figure><img src=".gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>
