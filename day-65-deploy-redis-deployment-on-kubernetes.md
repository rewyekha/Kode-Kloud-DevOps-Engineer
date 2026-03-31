# Day 65: Deploy Redis Deployment on Kubernetes

## Kubernetes: Deploy Redis with ConfigMap for In-Memory Caching

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Kubernetes, Redis, ConfigMap, Deployments, Volumes, emptyDir

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Create the ConfigMap](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-create-the-configmap)
   * [Step 2: Verify the ConfigMap](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-verify-the-configmap)
   * [Step 3: Create the Redis Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-create-the-redis-deployment)
   * [Step 4: Verify the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-the-deployment)
   * [Step 5: Verify the Pod is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-verify-the-pod-is-running)
4. [Complete Manifest Files](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#complete-manifest-files)
5. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus application development team observed performance issues with one of the applications deployed in the Kubernetes cluster. After investigation, the team decided to use **Redis** as an in-memory caching utility for the DB service. Redis will be deployed on the Kubernetes cluster for testing before moving to production.

**Requirements:**

1. Create a **ConfigMap** called `my-redis-config` having `maxmemory 2mb` in `redis-config`.
2. The **Deployment** should be named `redis-deployment`, using `redis:alpine` image with container named `redis-container`. It should have only `1` replica.
3. The container should request `1` CPU.
4. Mount `2` volumes:
   * An **EmptyDir** volume called `data` at path `/redis-master-data`
   * A **ConfigMap** volume called `redis-config` at path `/redis-master`
5. The container should expose port `6379`.
6. The `redis-deployment` should be up and running.

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

> **Target:** All `kubectl` commands are executed from `jump-host`.

***

### Solution

#### Step 1: Create the ConfigMap

Create the ConfigMap manifest using a heredoc and apply it to the cluster. The ConfigMap stores the Redis memory configuration as a key named `redis-config`:

```bash
cat <<EOF > my-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
EOF

kubectl apply -f my-redis-config.yaml
```

**Terminal Output:**

```bash
thor@jump-host ~$ cat <<EOF > my-redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
EOF
thor@jump-host ~$ kubectl apply -f my-redis-config.yaml
configmap/my-redis-config created
```

***

#### Step 2: Verify the ConfigMap

Confirm the ConfigMap exists and contains the correct data:

```bash
kubectl get configmap my-redis-config
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get configmap my-redis-config
NAME              DATA   AGE
my-redis-config   1      36s
```

Inspect the full ConfigMap contents:

```bash
kubectl describe configmap my-redis-config
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl describe configmap my-redis-config
Name:         my-redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
redis-config:
----
maxmemory 2mb

BinaryData
====
Events:  <none>
```

The ConfigMap `my-redis-config` is confirmed with one data key `redis-config` containing `maxmemory 2mb`.

***

#### Step 3: Create the Redis Deployment

Create the deployment manifest with both volumes, CPU request, and port configuration:

```bash
cat <<EOF > redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
EOF

kubectl apply -f redis-deployment.yaml
```

**Terminal Output:**

```bash
thor@jump-host ~$ cat <<EOF > redis-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
EOF
thor@jump-host ~$ kubectl apply -f redis-deployment.yaml
deployment.apps/redis-deployment created
```

***

#### Step 4: Verify the Deployment

Check the deployment status to confirm it is ready:

```bash
kubectl get deployment redis-deployment
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get deployment redis-deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
redis-deployment   1/1     1            1           16s
```

The deployment shows `READY: 1/1`, `UP-TO-DATE: 1`, and `AVAILABLE: 1` confirming the Redis container started successfully.

Also confirm the ConfigMap is still intact after deployment:

```bash
kubectl get configmap my-redis-config
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get configmap my-redis-config
NAME              DATA   AGE
my-redis-config   1      113s
```

***

#### Step 5: Verify the Pod is Running

List the pods managed by the deployment using its label selector:

```bash
kubectl get pods -l app=redis
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pods -l app=redis
NAME                               READY   STATUS    RESTARTS   AGE
redis-deployment-c795495f4-fqjwn   1/1     Running   0          24s
thor@jump-host ~$
```

The pod `redis-deployment-c795495f4-fqjwn` is `1/1 Running` with `0` restarts — confirming the Redis deployment is fully operational.

***

### Complete Manifest Files

#### my-redis-config.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
```

#### redis-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config
```

***

### Lab Complete

| Requirement     | Detail                                        | Status    |
| --------------- | --------------------------------------------- | --------- |
| ConfigMap name  | `my-redis-config`                             | Confirmed |
| ConfigMap key   | `redis-config`                                | Confirmed |
| ConfigMap value | `maxmemory 2mb`                               | Confirmed |
| Deployment name | `redis-deployment`                            | Confirmed |
| Image           | `redis:alpine`                                | Confirmed |
| Container name  | `redis-container`                             | Confirmed |
| Replicas        | `1`                                           | Confirmed |
| CPU request     | `1`                                           | Confirmed |
| Volume 1        | `data` (emptyDir) at `/redis-master-data`     | Confirmed |
| Volume 2        | `redis-config` (ConfigMap) at `/redis-master` | Confirmed |
| Container port  | `6379`                                        | Confirmed |
| Pod status      | `1/1 Running`, `0 restarts`                   | Confirmed |

***

### Key Concepts

#### ConfigMap as a Volume

In this lab, the ConfigMap is mounted as a **volume** rather than injected as environment variables. When a ConfigMap is mounted as a volume, each key in the ConfigMap becomes a file inside the container:

```bash
ConfigMap: my-redis-config
  key: redis-config
  value: maxmemory 2mb

Mounted at: /redis-master

Result inside container:
  /redis-master/redis-config   ← file containing "maxmemory 2mb"
```

Redis can then be started with `--include /redis-master/redis-config` or the config can be referenced directly, making configuration changes possible without rebuilding the image.

#### Two Volume Types Used

| Volume         | Type        | Mount Path           | Purpose                                           |
| -------------- | ----------- | -------------------- | ------------------------------------------------- |
| `data`         | `emptyDir`  | `/redis-master-data` | Temporary Redis data storage — lives with the pod |
| `redis-config` | `configMap` | `/redis-master`      | Injects Redis config file from ConfigMap          |

`emptyDir` volumes are created fresh when a pod starts and deleted when the pod is removed. For Redis in a testing scenario this is acceptable — in production, a `PersistentVolumeClaim` would be used to retain data across pod restarts.

#### CPU Resource Requests vs Limits

The deployment specifies a CPU **request** of `1` core:

```yaml
resources:
  requests:
    cpu: "1"
```

| Field      | Behaviour                                                                                                          |
| ---------- | ------------------------------------------------------------------------------------------------------------------ |
| `requests` | Minimum CPU guaranteed by the scheduler. The pod is only scheduled on a node with at least 1 CPU available.        |
| `limits`   | Maximum CPU the container can use. Not set here — container can burst beyond 1 CPU if the node has spare capacity. |

Setting only `requests` without `limits` is common in testing environments to ensure the pod gets scheduled with adequate resources while not artificially capping performance.

#### Why `redis:alpine`

The `alpine` variant of the Redis image is based on Alpine Linux — a minimal Linux distribution:

| Image          | Approximate Size | Use Case                         |
| -------------- | ---------------- | -------------------------------- |
| `redis:latest` | \~130MB          | Full-featured, Debian-based      |
| `redis:alpine` | \~30MB           | Lightweight, production-friendly |

`redis:alpine` is preferred for Kubernetes deployments because smaller images pull faster, consume less storage, and have a smaller attack surface.

#### Redis Default Port 6379

Redis always listens on port `6379` by default. The `containerPort: 6379` declaration in the Deployment is informational — it documents which port the container uses but does not actually open or restrict the port. Actual traffic routing requires a Kubernetes **Service** to expose the deployment to other pods or external clients.

***

_Lab completed on 2026-03-31 | Cluster: Kubernetes | Namespace: default | Pod: redis-deployment-c795495f4-fqjwn_

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>
