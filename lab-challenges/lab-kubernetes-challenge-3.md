# Lab- Kubernetes Challenge 3

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



## Deploying the Example Voting Application on Kubernetes

### Overview

This lab covers the deployment of a multi-tier voting application on Kubernetes within a dedicated namespace called `vote`. The application is based on the Docker Example Voting App and consists of five microservices: a voting frontend, a results frontend, a worker backend, a Redis cache, and a PostgreSQL database.

***

### Prerequisites

* A running Kubernetes cluster with `kubectl` configured
* Sufficient cluster resources to run five pods simultaneously
* Internet access from the cluster nodes to pull images from Docker Hub and the Docker samples registry

***

### Architecture Summary

The application is composed of five deployments and five services, all deployed in the `vote` namespace.

| Component | Type                 | Image                                  | Service Type | Port                       |
| --------- | -------------------- | -------------------------------------- | ------------ | -------------------------- |
| vote      | Deployment + Service | dockersamples/examplevotingapp\_vote   | NodePort     | 8080 -> 80, NodePort 31000 |
| result    | Deployment + Service | dockersamples/examplevotingapp\_result | NodePort     | 8081 -> 80, NodePort 31001 |
| worker    | Deployment           | dockersamples/examplevotingapp\_worker | None         | —                          |
| redis     | Deployment + Service | redis:alpine                           | ClusterIP    | 6379                       |
| db        | Deployment + Service | postgres:15-alpine                     | ClusterIP    | 5432                       |

**Data flow:**

1. A user submits a vote through the `vote` frontend (NodePort 31000).
2. The vote is stored in the `redis` cache.
3. The `worker` service reads from `redis` and writes the result to the `db` PostgreSQL database.
4. The `result` frontend (NodePort 31001) reads from `db` and displays the live tally.

***

### Step 1: Create the Namespace

All resources in this lab are deployed into a dedicated namespace named `vote`.

```bash
kubectl create namespace vote
```

***

### Step 2: Deploy Redis

Redis acts as the message queue between the vote frontend and the worker. It uses an `emptyDir` volume to store data at `/data`.

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
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
      volumes:
        - name: redis-data
          emptyDir: {}
      containers:
        - name: redis
          image: redis:alpine
          volumeMounts:
            - name: redis-data
              mountPath: /data
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  type: ClusterIP
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

Apply both:

```bash
kubectl apply -n vote -f redis-deployment.yaml
kubectl apply -n vote -f redis-service.yaml
```

***

### Step 3: Deploy PostgreSQL (db)

The `db` deployment runs PostgreSQL 15 and stores the final vote tallies. It uses `POSTGRES_HOST_AUTH_METHOD=trust` to allow passwordless connections, and an `emptyDir` volume for data storage.

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      volumes:
        - name: db-data
          emptyDir: {}
      containers:
        - name: postgres
          image: postgres:15-alpine
          env:
            - name: POSTGRES_HOST_AUTH_METHOD
              value: trust
          volumeMounts:
            - name: db-data
              mountPath: /var/lib/postgresql/data
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  type: ClusterIP
  selector:
    app: db
  ports:
    - port: 5432
      targetPort: 5432
```

Apply both:

```bash
kubectl apply -n vote -f db-deployment.yaml
kubectl apply -n vote -f db-service.yaml
```

***

### Step 4: Deploy the Vote Frontend

The `vote` deployment runs the voting interface. It is exposed externally via a NodePort service on port 31000.

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vote
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vote
  template:
    metadata:
      labels:
        app: vote
    spec:
      containers:
        - name: vote
          image: dockersamples/examplevotingapp_vote
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: vote
spec:
  type: NodePort
  selector:
    app: vote
  ports:
    - port: 8080
      targetPort: 80
      nodePort: 31000
```

Apply both:

```bash
kubectl apply -n vote -f vote-deployment.yaml
kubectl apply -n vote -f vote-service.yaml
```

The voting interface is accessible at `http://<node-ip>:31000`.

***

### Step 5: Deploy the Worker

The worker service reads votes from Redis and persists them to PostgreSQL. It requires no service definition as it does not receive inbound traffic.

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: worker
spec:
  replicas: 1
  selector:
    matchLabels:
      app: worker
  template:
    metadata:
      labels:
        app: worker
    spec:
      containers:
        - name: worker
          image: dockersamples/examplevotingapp_worker
```

Apply:

```bash
kubectl apply -n vote -f worker-deployment.yaml
```

***

### Step 6: Deploy the Result Frontend

The `result` deployment displays the live vote tally. It is exposed externally via a NodePort service on port 31001.

#### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: result
spec:
  replicas: 1
  selector:
    matchLabels:
      app: result
  template:
    metadata:
      labels:
        app: result
    spec:
      containers:
        - name: result
          image: dockersamples/examplevotingapp_result
```

#### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: result
spec:
  type: NodePort
  selector:
    app: result
  ports:
    - port: 8081
      targetPort: 80
      nodePort: 31001
```

Apply both:

```bash
kubectl apply -n vote -f result-deployment.yaml
kubectl apply -n vote -f result-service.yaml
```

The results interface is accessible at `http://<node-ip>:31001`.

***

### Verification

Check the status of all resources in the `vote` namespace:

```bash
kubectl get all -n vote
```

Expected output:

```bash
NAME                          READY   STATUS    RESTARTS   AGE
pod/db-bc7f4bbf5-vvt4b        1/1     Running   0          ...
pod/redis-6c48d68585-c2k9z    1/1     Running   0          ...
pod/result-7996c8c8f4-sgf75   1/1     Running   0          ...
pod/vote-5fdbfc876f-2k4hc     1/1     Running   0          ...
pod/worker-5dd767667f-lzgv2   1/1     Running   0          ...

NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/db       ClusterIP   172.20.99.69     <none>        5432/TCP         ...
service/redis    ClusterIP   172.20.249.119   <none>        6379/TCP         ...
service/result   NodePort    172.20.106.115   <none>        8081:31001/TCP   ...
service/vote     NodePort    172.20.110.185   <none>        8080:31000/TCP   ...

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db       1/1     1            1           ...
deployment.apps/redis    1/1     1            1           ...
deployment.apps/result   1/1     1            1           ...
deployment.apps/vote     1/1     1            1           ...
deployment.apps/worker   1/1     1            1           ...
```

All five pods must be in `Running` state and all five deployments must show `1/1` available before the application functions correctly.

***

### Notes and Observations

**emptyDir volumes:** Both `redis` and `db` use `emptyDir` volumes. This means data is stored only for the lifetime of the pod. Restarting or rescheduling either pod will result in data loss. For production workloads, these should be replaced with PersistentVolumeClaims backed by durable storage.

**POSTGRES\_HOST\_AUTH\_METHOD=trust:** This environment variable configures PostgreSQL to accept all connections without a password. This is appropriate for isolated lab environments but must never be used in production. Use a proper secret-backed password instead.

**Service naming and DNS:** Kubernetes creates DNS entries for each service within the namespace. The `vote`, `worker`, and `result` applications connect to `redis` and `db` using their service names as hostnames. This is why the service names must exactly match what the application code expects. Changing a service name will cause connection failures.

**Worker has no service:** The `worker` deployment does not need a Kubernetes Service because it only initiates outbound connections to `redis` and `db`. It does not receive inbound traffic from any other component.

**NodePort service port mapping:** The `vote` service maps external port 31000 to internal port 8080, which then forwards to container port 80. The `result` service maps external port 31001 to internal port 8081, which also forwards to container port 80. The intermediate `port` field in each service definition allows other in-cluster clients to reach the service at a distinct port if needed.

***

### Resource Summary

| Resource   | Name   | Namespace | External Access |
| ---------- | ------ | --------- | --------------- |
| Deployment | vote   | vote      | NodePort 31000  |
| Deployment | result | vote      | NodePort 31001  |
| Deployment | worker | vote      | None            |
| Deployment | redis  | vote      | ClusterIP only  |
| Deployment | db     | vote      | ClusterIP only  |
| Service    | vote   | vote      | NodePort 31000  |
| Service    | result | vote      | NodePort 31001  |
| Service    | redis  | vote      | ClusterIP 6379  |
| Service    | db     | vote      | ClusterIP 5432  |

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
