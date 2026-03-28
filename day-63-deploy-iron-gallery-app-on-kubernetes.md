# Day 63: Deploy Iron Gallery App on Kubernetes

## Kubernetes: Deploy Iron Gallery Application with Namespace, Deployments, and Services

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Kubernetes, Namespaces, Deployments, Services, NodePort, ClusterIP, emptyDir

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Create the Namespace](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-create-the-namespace)
   * [Step 2: Create the Iron Gallery Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-create-the-iron-gallery-deployment)
   * [Step 3: Create the Iron DB Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-create-the-iron-db-deployment)
   * [Step 4: Create the Iron DB Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-create-the-iron-db-service)
   * [Step 5: Create the Iron Gallery Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-the-iron-gallery-service)
   * [Step 6: Verify All Resources](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-verify-all-resources)
   * [Step 7: Verify Pods are Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-verify-pods-are-running)
4. [Complete Manifest Files](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#complete-manifest-files)
5. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus DevOps team has customized the Iron Gallery app and is deploying it on a Kubernetes cluster.

**Requirements:**

**1. Create a namespace** `iron-namespace-datacenter`

**2. Create a deployment** `iron-gallery-deployment-datacenter` for Iron Gallery:

* Labels `run` should be `iron-gallery`
* Replicas count: `1`
* Selector matchLabels `run`: `iron-gallery`
* Template labels `run`: `iron-gallery`
* Container name: `iron-gallery-container-datacenter`, image: `kodekloud/irongallery:2.0`
* Resource limits: memory `100Mi`, cpu `50m`
* VolumeMount 1: name `config`, mountPath `/usr/share/nginx/html/data`
* VolumeMount 2: name `images`, mountPath `/usr/share/nginx/html/uploads`
* Volume 1: name `config`, type `emptyDir`
* Volume 2: name `images`, type `emptyDir`

**3. Create a deployment** `iron-db-deployment-datacenter` for Iron DB:

* Labels `db` should be `mariadb`
* Replicas count: `1`
* Selector matchLabels `db`: `mariadb`
* Template labels `db`: `mariadb`
* Container name: `iron-db-container-datacenter`, image: `kodekloud/irondb:2.0`
* Environment variables: `MYSQL_DATABASE=database_host`, `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD`, `MYSQL_USER` (non-root)
* VolumeMount: name `db`, mountPath `/var/lib/mysql`
* Volume: name `db`, type `emptyDir`

**4. Create a service** `iron-db-service-datacenter`:

* Selector `db: mariadb`
* Protocol TCP, port/targetPort `3306`
* Type: `ClusterIP`

**5. Create a service** `iron-gallery-service-datacenter`:

* Selector `run: iron-gallery`
* Protocol TCP, port/targetPort `80`, nodePort `32678`
* Type: `NodePort`

> **Notes:**
>
> * No connection between database and frontend is required — if the installation page appears, that is sufficient.
> * `kubectl` on `jump-host` is pre-configured to work with the cluster.

***

### Infrastructure Details

> **Target:** All `kubectl` commands are executed from `jump-host`.

***

### Solution

#### Step 1: Create the Namespace

Create the dedicated namespace to isolate all Iron Gallery resources:

```bash
kubectl create namespace iron-namespace-datacenter
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl create namespace iron-namespace-datacenter
namespace/iron-namespace-datacenter created
```

***

#### Step 2: Create the Iron Gallery Deployment

Create the manifest file for the frontend web application deployment:

```bash
vi iron-gallery-deployment.yaml
```

Apply the manifest:

```bash
kubectl apply -f iron-gallery-deployment.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ vi iron-gallery-deployment.yaml
thor@jump-host ~$ kubectl apply -f iron-gallery-deployment.yaml
```

The deployment `iron-gallery-deployment-datacenter` was created in the `iron-namespace-datacenter` namespace.

***

#### Step 3: Create the Iron DB Deployment

Create the manifest file for the MariaDB database deployment:

```bash
vi iron-db-deployment.yaml
kubectl apply -f iron-db-deployment.yaml
```

The deployment `iron-db-deployment-datacenter` was created with all required environment variables for the MariaDB instance.

***

#### Step 4: Create the Iron DB Service

Create the ClusterIP service to expose the database internally within the cluster:

```bash
vi iron-db-service.yaml
kubectl apply -f iron-db-service.yaml
```

***

#### Step 5: Create the Iron Gallery Service

Create the NodePort service to expose the Iron Gallery web application externally:

```bash
vi iron-gallery-service.yaml
kubectl apply -f iron-gallery-service.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ vi iron-gallery-service.yaml
thor@jump-host ~$ kubectl apply -f iron-gallery-service.yaml
service/iron-gallery-service-datacenter created
```

***

#### Step 6: Verify All Resources

Check all resources created in the namespace:

```bash
kubectl get all -n iron-namespace-datacenter
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get all -n iron-namespace-datacenter
NAME                                                      READY   STATUS    RESTARTS   AGE
pod/iron-db-deployment-datacenter-6d59bdfc8b-sqdf6        1/1     Running   0          81s
pod/iron-gallery-deployment-datacenter-7f74bfd494-5gfm9   1/1     Running   0          2m49s

NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/iron-db-service-datacenter        ClusterIP   10.43.152.222   <none>        3306/TCP       51s
service/iron-gallery-service-datacenter   NodePort    10.43.102.26    <none>        80:32678/TCP   7s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/iron-db-deployment-datacenter        1/1     1            1           81s
deployment.apps/iron-gallery-deployment-datacenter   1/1     1            1           2m49s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/iron-db-deployment-datacenter-6d59bdfc8b        1         1         1       81s
replicaset.apps/iron-gallery-deployment-datacenter-7f74bfd494   1         1         1       2m49s
```

All resources are confirmed healthy:

* Both pods: `1/1 Running`
* `iron-db-service-datacenter`: `ClusterIP` on port `3306`
* `iron-gallery-service-datacenter`: `NodePort` on `80:32678`
* Both deployments: `1/1 Ready`

***

#### Step 7: Verify Pods are Running

```bash
kubectl get pods -n iron-namespace-datacenter
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get pods -n iron-namespace-datacenter
NAME                                                  READY   STATUS    RESTARTS   AGE
iron-db-deployment-datacenter-6d59bdfc8b-sqdf6        1/1     Running   0          89s
iron-gallery-deployment-datacenter-7f74bfd494-5gfm9   1/1     Running   0          2m57s
```

Both pods are `Running` with `0` restarts.

Check cluster node details:

```bash
kubectl get nodes -o wide
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get nodes -o wide
NAME        STATUS   ROLES           AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
jump-host   Ready    control-plane   46m   v1.34.1+k3s1   10.244.247.246   <none>        Alpine Linux v3.16   6.8.0-90-generic   containerd://1.6.8
```

***

### Complete Manifest Files

#### iron-gallery-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-datacenter
  namespace: iron-namespace-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
      - name: iron-gallery-container-datacenter
        image: kodekloud/irongallery:2.0
        resources:
          limits:
            memory: "100Mi"
            cpu: "50m"
        volumeMounts:
        - name: config
          mountPath: /usr/share/nginx/html/data
        - name: images
          mountPath: /usr/share/nginx/html/uploads
      volumes:
      - name: config
        emptyDir: {}
      - name: images
        emptyDir: {}
```

#### iron-db-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-datacenter
  namespace: iron-namespace-datacenter
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
      - name: iron-db-container-datacenter
        image: kodekloud/irondb:2.0
        env:
        - name: MYSQL_DATABASE
          value: database_host
        - name: MYSQL_ROOT_PASSWORD
          value: StrongRootPass@123
        - name: MYSQL_PASSWORD
          value: StrongUserPass@123
        - name: MYSQL_USER
          value: dbuser
        volumeMounts:
        - name: db
          mountPath: /var/lib/mysql
      volumes:
      - name: db
        emptyDir: {}
```

#### iron-db-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  selector:
    db: mariadb
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
  type: ClusterIP
```

#### iron-gallery-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-datacenter
  namespace: iron-namespace-datacenter
spec:
  selector:
    run: iron-gallery
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 32678
  type: NodePort
```

***

### Lab Complete

| Requirement        | Resource                             | Configuration                          | Status    |
| ------------------ | ------------------------------------ | -------------------------------------- | --------- |
| Namespace          | `iron-namespace-datacenter`          | Created                                | Confirmed |
| Gallery Deployment | `iron-gallery-deployment-datacenter` | 1 replica, `kodekloud/irongallery:2.0` | Running   |
| Gallery Container  | `iron-gallery-container-datacenter`  | Memory 100Mi, CPU 50m                  | Confirmed |
| Gallery Volumes    | `config` + `images`                  | emptyDir at correct mount paths        | Confirmed |
| DB Deployment      | `iron-db-deployment-datacenter`      | 1 replica, `kodekloud/irondb:2.0`      | Running   |
| DB Container       | `iron-db-container-datacenter`       | All 4 env vars set                     | Confirmed |
| DB Volume          | `db`                                 | emptyDir at `/var/lib/mysql`           | Confirmed |
| DB Service         | `iron-db-service-datacenter`         | ClusterIP, port 3306                   | Confirmed |
| Gallery Service    | `iron-gallery-service-datacenter`    | NodePort 32678 → 80                    | Confirmed |
| All pods           | Both pods                            | `1/1 Running`, `0 restarts`            | Confirmed |
| App accessible     | Port 32678                           | Installation page displayed            | Confirmed |

***

### Key Concepts

#### Namespace Isolation

All five resources — two deployments, two services, and their pods — were created inside the `iron-namespace-datacenter` namespace. Namespaces provide logical separation within a Kubernetes cluster:

```bash
Cluster
└── iron-namespace-datacenter
    ├── Deployment: iron-gallery-deployment-datacenter
    ├── Deployment: iron-db-deployment-datacenter
    ├── Service: iron-gallery-service-datacenter (NodePort)
    ├── Service: iron-db-service-datacenter (ClusterIP)
    └── Pods (managed by ReplicaSets)
```

Every `kubectl` command targeting these resources must include `-n iron-namespace-datacenter` or the resources will not be visible.

#### ClusterIP vs NodePort Services

| Service Type | Accessible From                        | Use Case                                  |
| ------------ | -------------------------------------- | ----------------------------------------- |
| `ClusterIP`  | Inside the cluster only                | Internal service-to-service communication |
| `NodePort`   | Outside the cluster via node IP + port | External access to web applications       |

`iron-db-service-datacenter` uses `ClusterIP` because the database should only be reachable by the gallery application inside the cluster — not from the internet. `iron-gallery-service-datacenter` uses `NodePort 32678` to expose the web interface externally.

#### Why the Installation Page is Expected

The Iron Gallery app showed a database installation/configuration page when accessed via NodePort. This is correct and expected behaviour based on the lab note:

> "We don't need to make connection b/w database and front-end now, if the installation page is coming up it should be enough for now."

The installation page appearing confirms:

* The `iron-gallery` pod is running and serving HTTP traffic
* The NodePort `32678` is correctly forwarding traffic to container port `80`
* Nginx is running inside the container

The page asks for database connection details because the application has not yet been configured to point to `iron-db-service-datacenter`. That connection would be established in a subsequent task by providing the service DNS name `iron-db-service-datacenter` and the credentials set in the deployment environment variables.

#### emptyDir Volumes

Both deployments use `emptyDir` volumes. An `emptyDir` volume is created when a pod is assigned to a node and exists only for the lifetime of that pod:

| Volume   | Mount Path                      | Purpose                         |
| -------- | ------------------------------- | ------------------------------- |
| `config` | `/usr/share/nginx/html/data`    | Iron Gallery configuration data |
| `images` | `/usr/share/nginx/html/uploads` | Uploaded image files            |
| `db`     | `/var/lib/mysql`                | MariaDB data files              |

For the Iron DB deployment, the `emptyDir` at `/var/lib/mysql` means the database data does not persist across pod restarts — suitable for this initial deployment and testing phase.

#### Label Selectors Connecting Services to Pods

Services use label selectors to find their target pods. The Iron Gallery and Iron DB use different label keys to avoid any cross-selection:

```bash
iron-gallery pods:          label: run=iron-gallery
iron-gallery service:       selector: run: iron-gallery  ← matches

iron-db pods:               label: db=mariadb
iron-db service:            selector: db: mariadb        ← matches
```

Using separate label keys (`run` for the gallery, `db` for the database) is a deliberate design choice that prevents any possibility of a service accidentally selecting the wrong pod type.

***

_Lab completed on 2026-03-27 | Cluster: Kubernetes (k3s v1.34.1) | Namespace: iron-namespace-datacenter_

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
