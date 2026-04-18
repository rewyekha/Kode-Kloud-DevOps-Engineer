# Day 67: Deploy Guest Book App on Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

### Overview

The Nautilus Application development team has finished development of one of the applications and it is ready for deployment. It is a guestbook application that will be used to manage entries for guests/visitors. As per discussion with the DevOps team, they have finalized the infrastructure that will be deployed on Kubernetes cluster. Below you can find more details about it.

***

### Back-End Tier

#### Redis Master

Create a deployment named `redis-master` for Redis master.

1. **Replicas count:** 1
2. **Container name:** `master-redis-devops` and it should use image `redis`.
3. **Request resources:** CPU should be 100m and Memory should be 100Mi.
4. **Container port:** Redis default port i.e 6379.

Create a service named `redis-master` for Redis master.

* **Port and targetPort:** Redis default port i.e 6379.

***

#### Redis Slave

Create another deployment named `redis-slave` for Redis slave.

1. **Replicas count:** 2
2. **Container name:** `slave-redis-devops` and it should use `gcr.io/google_samples/gb-redisslave:v3` image.
3. **Request resources:** CPU should be 100m and Memory should be 100Mi.
4. **Environment variable:** Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`.
5. **Container port:** Redis default port i.e 6379.

Create another service named `redis-slave`.

* **Port:** Redis default port i.e 6379.

***

### Front-End Tier

#### Frontend Deployment

Create a deployment named `frontend`.

1. **Replicas count:** 3
2. **Container name:** `php-redis-devops` and it should use image `gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff`.
3. **Request resources:** CPU should be 100m and Memory should be 100Mi.
4. **Environment variable:** Define an environment variable named `GET_HOSTS_FROM` and its value should be `dns`.
5. **Container port:** 80

Create a service named `frontend`.

* **Type:** NodePort
* **Port:** 80
* **NodePort:** 30009

***

### Additional Notes

* You can use any labels as per your choice.
* The `kubectl` utility on the jump-host has been configured to work with the Kubernetes cluster.
* After deployment, you can check the guestbook app by clicking on the App button.

***

## Nautilus Guestbook Application Deployment on Kubernetes

### Overview

The **Nautilus Guestbook Application** is designed to manage entries for guests and visitors. The application consists of a **frontend tier** and a **backend Redis tier**, which includes both a **master** and **slave** configuration. The infrastructure is deployed on a Kubernetes cluster.


***

### Architecture

The deployment is organized into two tiers:

#### Backend Tier (Redis)

1. **Redis Master**
   * Single replica
   * Handles write operations
   * Exposes port `6379`
   * ClusterIP service for internal access
2. **Redis Slave**
   * Two replicas
   * Replicates data from the master
   * Exposes port `6379`
   * ClusterIP service for internal access
   * Environment variable `GET_HOSTS_FROM=dns` for dynamic service discovery

#### Frontend Tier

* Three replicas
* PHP-based frontend connecting to the Redis tier
* Exposes port `80` externally through a NodePort service (`30009`)
* Environment variable `GET_HOSTS_FROM=dns` for service discovery

***

### Prerequisites

Before deploying, ensure the following:

* Access to a Kubernetes cluster
* `kubectl` configured to communicate with the cluster
* Network access to the cluster nodes for external access to the frontend NodePort

***

### Deployment Details

#### Redis Master

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: master
  template:
    metadata:
      labels:
        app: redis
        role: master
    spec:
      containers:
      - name: master-redis-devops
        image: redis
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: master
```

***

#### Redis Slave

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
      role: slave
  template:
    metadata:
      labels:
        app: redis
        role: slave
    spec:
      containers:
      - name: slave-redis-devops
        image: gcr.io/google_samples/gb-redisslave:v3
        ports:
        - containerPort: 6379
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: slave
```

***

#### Frontend

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis-devops
        image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
        ports:
        - containerPort: 80
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30009
  selector:
    app: guestbook
    tier: frontend
```

***

### Deployment Commands

Apply each YAML file using `kubectl`:

```bash
kubectl apply -f redis-master-deployment.yaml
kubectl apply -f redis-master-service.yaml
kubectl apply -f redis-slave-deployment.yaml
kubectl apply -f redis-slave-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

***

### Verification

Check the status of deployments:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get svc
```

Verify container names and images:

```bash
kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{': '}{.spec.containers[0].name}{'\n'}{end}"
kubectl get pods -o jsonpath="{range .items[*]}{.metadata.name}{': '}{.spec.containers[0].image}{'\n'}{end}"
```

***

### Accessing the Application

Once the services are running, access the frontend application via:

```
http://<Node-IP>:30009
```

Replace `<Node-IP>` with the IP of your Kubernetes node where the frontend NodePort service is available.

***

### Notes

* All backend Redis services use **ClusterIP** as they are intended for internal communication.
* The frontend is exposed externally via a **NodePort** for user access.
* Resource requests ensure the pods have minimum CPU and memory allocations.
* Environment variable `GET_HOSTS_FROM=dns` allows pods to discover service endpoints via Kubernetes DNS.

***

**Before**

<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

**Final:**<br>

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
