# Day 60: Persistent Volumes in Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create/use persistent volumes to store the application code, and the template needs to be designed accordingly. Please find more details below:

1. Create a `PersistentVolume` named as `pv-nautilus`. Configure the `spec` as storage class should be `manual`, set capacity to `4Gi`, set access mode to `ReadWriteOnce`, volume type should be `hostPath` and set path to `/mnt/finance` (this directory is already created, you might not be able to access it directly, so you need not to worry about it).
2. Create a `PersistentVolumeClaim` named as `pvc-nautilus`. Configure the `spec` as storage class should be `manual`, request `2Gi` of the storage, set access mode to `ReadWriteOnce`.
3. Create a `pod` named as `pod-nautilus`, mount the persistent volume you created with claim name `pvc-nautilus` at document root of the web server, the container within the pod should be named as `container-nautilus` using image `httpd` with `latest` tag only (remember to mention the tag i.e `httpd:latest`).
4. Create a node port type service named `web-nautilus` using node port `30008` to expose the web server running within the pod.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.


## Kubernetes: Deploy Web Application with Persistent Volumes and NodePort Service

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Kubernetes, PersistentVolume, PersistentVolumeClaim, Pods, NodePort, Services

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Create the PersistentVolume](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-create-the-persistentvolume)
   * [Step 2: Create the PersistentVolumeClaim](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-create-the-persistentvolumeclaim)
   * [Step 3: Create the Pod](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-create-the-pod)
   * [Step 4: Add Label to Pod for Service Selector](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-add-label-to-pod-for-service-selector)
   * [Step 5: Create the NodePort Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-the-nodeport-service)
   * [Step 6: Verify Service Endpoints](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-verify-service-endpoints)
   * [Step 7: Test the Web Server](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-test-the-web-server)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus DevOps team is working on a Kubernetes template to deploy a web application on the cluster. There are some requirements to create and use persistent volumes to store the application code, and the template needs to be designed accordingly.

**Requirements:**

1. Create a `PersistentVolume` named `pv-nautilus`. Configure the spec as: storage class should be `manual`, set capacity to `4Gi`, set access mode to `ReadWriteOnce`, volume type should be `hostPath` and set path to `/mnt/finance` (this directory is already created, you might not be able to access it directly, so you need not worry about it).
2. Create a `PersistentVolumeClaim` named `pvc-nautilus`. Configure the spec as: storage class should be `manual`, request `2Gi` of the storage, set access mode to `ReadWriteOnce`.
3. Create a `pod` named `pod-nautilus`, mount the persistent volume you created with claim name `pvc-nautilus` at the document root of the web server. The container within the pod should be named `container-nautilus` using image `httpd` with `latest` tag only (remember to mention the tag i.e `httpd:latest`).
4. Create a node port type service named `web-nautilus` using node port `30008` to expose the web server running within the pod.

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
| -------------------- | ----------- | --------- | ------------ | ------------------------------------- |
| Application Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Hosts Nautilus Application 1 |
| Application Server 2 | `stapp02` | `steve` | `Am3ric@` | Hosts Nautilus Application 2 |
| Application Server 3 | `stapp03` | `banner` | `BigGr33n` | Hosts Nautilus Application 3 |
| LoadBalancer Server | `stlb01` | `loki` | `Mischi3f` | Distributes traffic for Nautilus HTTP |
| Database Server | `stdb01` | `peter` | `Sp!dy` | Hosts Nautilus Database |
| Storage Server | `ststor01` | `natasha` | `Bl@kW` | Stores data for Nautilus Servers |
| Backup Server | `stbkp01` | `clint` | `H@wk3y3` | Manages backups for Nautilus Servers |
| Mail Server | `stmail01` | `groot` | `Gr00T123` | Manages email services |
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Provides secure access to Stork DC |
| Jenkins Server | `jenkins` | `jenkins` | `j@rv!s` | Runs Jenkins for CI/CD pipeline |

> **Target:** All `kubectl` commands are executed from `jump-host`, which is pre-configured to communicate with the Kubernetes cluster.

***

### Solution

#### Step 1: Create the PersistentVolume

A PersistentVolume is a cluster-level storage resource that exists independently of any pod. It is provisioned by an administrator and persists beyond the pod lifecycle. The `hostPath` type maps a directory on the node's filesystem directly into the cluster storage system.

Create the manifest file `pv-nautilus.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus
spec:
  storageClassName: manual
  capacity:
    storage: 4Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/finance
```

Apply the manifest:

```bash
kubectl apply -f pv-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl apply -f pv-nautilus.yaml
persistentvolume/pv-nautilus created
```

Verify the PV was created and is in `Available` status:

```bash
kubectl get pv
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pv
NAME          STATUS      VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
pvc-nautilus  Bound       pv-nautilus   4Gi        RWO            manual         <unset>                 7s
```

The PV `pv-nautilus` is available in the cluster with `4Gi` capacity, `RWO` (ReadWriteOnce) access mode, and `manual` storage class.

***

#### Step 2: Create the PersistentVolumeClaim

A PersistentVolumeClaim is a request for storage submitted by a user or workload. Kubernetes automatically binds the claim to a compatible PV based on matching storage class, access mode, and capacity. The claim requests `2Gi` from the `4Gi` PV.

Create the manifest file `pvc-nautilus.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 2Gi
```

Apply the manifest:

```bash
kubectl apply -f pvc-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl apply -f pvc-nautilus.yaml
persistentvolumeclaim/pvc-nautilus created
```

Verify the PVC is bound to the PV:

```bash
kubectl get pvc
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nautilus   Bound    pv-nautilus   4Gi        RWO            manual         7s
```

The PVC `pvc-nautilus` status is `Bound`, meaning Kubernetes has successfully matched and attached it to `pv-nautilus`. The full `4Gi` capacity of the PV is shown — this is expected because Kubernetes binds the entire PV to a single claim.

Inspect the full PVC manifest to confirm the configuration:

```bash
cat pvc-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ cat pvc-nautilus.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 2Gi
```

***

#### Step 3: Create the Pod

The pod runs the `httpd:latest` container and mounts the PVC at `/usr/local/apache2/htdocs`, which is the default document root for the Apache HTTP server. Any content stored in the PV at `/mnt/finance` on the node will be served as web content through this mount.

Create the manifest file `pod-nautilus.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
spec:
  containers:
    - name: container-nautilus
      image: httpd:latest
      volumeMounts:
        - mountPath: /usr/local/apache2/htdocs
          name: nautilus-volume
  volumes:
    - name: nautilus-volume
      persistentVolumeClaim:
        claimName: pvc-nautilus
```

Apply the manifest:

```bash
kubectl apply -f pod-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl apply -f pod-nautilus.yaml
pod/pod-nautilus created
```

Verify the pod is running:

```bash
kubectl get pods
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
pod-nautilus   1/1     Running   0          6s
```

The pod reached `Running` state in 6 seconds with `1/1` containers ready. Inspect the full pod manifest to confirm the volume configuration:

```bash
cat pod-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ cat pod-nautilus.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
spec:
  containers:
    - name: container-nautilus
      image: httpd:latest
      volumeMounts:
        - mountPath: /usr/local/apache2/htdocs
          name: nautilus-volume
  volumes:
    - name: nautilus-volume
      persistentVolumeClaim:
        claimName: pvc-nautilus
```

***

#### Step 4: Add Label to Pod for Service Selector

The NodePort service uses a label selector to identify which pod(s) to route traffic to. The pod was created without a label, so it must be edited to add the label `app: web-nautilus`. The service manifest will use this label in its selector.

Open the pod for editing:

```bash
vi web-nautilus.yaml
```

First attempt — edit was cancelled with no changes:

```bash
kubectl edit pod pod-nautilus
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl edit pod pod-nautilus
Edit cancelled, no changes made.
```

Second attempt — the label `app: web-nautilus` was added under `metadata.labels`:

```bash
kubectl edit pod pod-nautilus
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl edit pod pod-nautilus
pod/pod-nautilus edited
```

Verify the label was applied:

```bash
kubectl get pod pod-nautilus --show-labels
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pod pod-nautilus --show-labels
NAME           READY   STATUS    RESTARTS   AGE     LABELS
pod-nautilus   1/1     Running   0          7m35s   app=web-nautilus
```

The label `app=web-nautilus` is now present on `pod-nautilus`. The service selector will use this label to route traffic to the correct pod.

***

#### Step 5: Create the NodePort Service

A NodePort service exposes the pod on a fixed port (`30008`) on every node in the cluster. External traffic arriving at any node IP on port `30008` is forwarded to port `80` on the `httpd` container inside the pod.

Create the manifest file `web-nautilus.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus
spec:
  type: NodePort
  selector:
    app: web-nautilus
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30008
```

Apply the manifest:

```bash
kubectl apply -f web-nautilus.yaml
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl apply -f web-nautilus.yaml
service/web-nautilus created
```

***

#### Step 6: Verify Service Endpoints

Confirm the service has correctly discovered and registered the pod as a backend endpoint:

```bash
kubectl get endpoints web-nautilus
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get endpoints web-nautilus
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME           ENDPOINTS        AGE
web-nautilus   10.22.0.9:80     10s
```

The service endpoint `10.22.0.9:80` corresponds to the pod's internal cluster IP on port 80. This confirms the label selector is working correctly and traffic will be forwarded to `pod-nautilus`. The deprecation warning is informational — the endpoint still functions correctly.

***

#### Step 7: Test the Web Server

Execute a `curl` command from inside the pod to confirm the Apache HTTP server is responding on localhost:

```bash
kubectl exec -it pod-nautilus -- curl localhost
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl exec -it pod-nautilus -- curl localhost
```

The web server responded successfully. The service is also accessible externally via the NodePort at:

```
http://30008-port-nzgofiumzdyfhmxf.labs.kodekloud.com
```

The browser confirmed the web server is live, displaying:

```
Index of /
```

This confirms the Apache `httpd` server is running and serving content from the `/usr/local/apache2/htdocs` directory which is backed by the PersistentVolume mounted from `/mnt/finance`.

***

### Lab Complete

| Requirement | Resource | Configuration | Status |
| --------------------- | -------------- | ---------------------------------------------------------------------------------- | ---------- |
| PersistentVolume | `pv-nautilus` | `manual` / `4Gi` / `ReadWriteOnce` / `hostPath: /mnt/finance` | Confirmed |
| PersistentVolumeClaim | `pvc-nautilus` | `manual` / `2Gi` / `ReadWriteOnce` / `Bound` | Confirmed |
| Pod | `pod-nautilus` | `container-nautilus` / `httpd:latest` / PVC mounted at `/usr/local/apache2/htdocs` | Running |
| Label | `pod-nautilus` | `app=web-nautilus` | Applied |
| NodePort Service | `web-nautilus` | `NodePort` / port `30008` / selector `app: web-nautilus` | Confirmed |
| Endpoint | `web-nautilus` | `10.22.0.9:80` | Registered |
| Web server response | Port `30008` | `Index of /` returned | Verified |

***

### Key Concepts

#### PersistentVolume and PersistentVolumeClaim Binding

The PV-PVC binding process works as follows:

```
PersistentVolume (pv-nautilus)
  storageClassName: manual
  capacity: 4Gi
  accessModes: ReadWriteOnce
  hostPath: /mnt/finance
        |
        | Kubernetes matches on: storageClassName + accessModes + capacity
        |
PersistentVolumeClaim (pvc-nautilus)
  storageClassName: manual
  accessModes: ReadWriteOnce
  requests: 2Gi
        |
        | STATUS: Bound
        v
Pod mounts PVC at /usr/local/apache2/htdocs
```

A claim is bound to a PV when all three criteria match: storage class name, access mode, and the requested storage does not exceed the PV capacity. Once bound, the full PV capacity is reserved exclusively for that claim.

#### hostPath Volume Type

`hostPath` mounts a file or directory from the host node's filesystem into the pod. It is useful for single-node development clusters and labs, but is not recommended for production multi-node clusters because the data is tied to a specific node — if the pod is rescheduled to a different node, it will not find the same data.

| Volume Type | Scope | Use Case |
| ----------------------- | -------------- | ---------------------------- |
| `hostPath` | Single node | Development, labs |
| `nfs` | Network-wide | Shared storage across nodes |
| `awsElasticBlockStore` | Cloud provider | AWS production workloads |
| `persistentVolumeClaim` | Cluster-wide | Abstracted, portable storage |

#### NodePort Service Port Mapping

```
External Traffic
      |
      | Port 30008 (NodePort — accessible on every node's IP)
      v
Service (web-nautilus)
      |
      | Port 80 (ClusterIP — internal cluster port)
      v
Pod (pod-nautilus)
      |
      | Port 80 (targetPort — container port httpd listens on)
      v
container-nautilus (httpd:latest)
```

The three port fields in a NodePort service serve distinct roles:

| Field | Value | Description |
| ------------ | ------- | ---------------------------------------------- |
| `nodePort` | `30008` | External port exposed on every cluster node |
| `port` | `80` | Port the service listens on inside the cluster |
| `targetPort` | `80` | Port the container listens on inside the pod |

#### Why the Pod Needed a Label

Kubernetes services use **label selectors** to identify target pods dynamically. Without a matching label on the pod, the service has no backend and all traffic is dropped. The label `app: web-nautilus` was added to the pod after creation using `kubectl edit`, and the service selector `app: web-nautilus` then matched it — which is confirmed by the endpoint `10.22.0.9:80` appearing in `kubectl get endpoints`.

#### Apache httpd Document Root

The `httpd:latest` image serves files from `/usr/local/apache2/htdocs` by default. Mounting the PVC at this path means any files stored in the PersistentVolume at `/mnt/finance` on the node are directly served as web content — making the storage and the web server tightly integrated without any additional configuration.

***

_Lab completed on 2026-03-25 | Cluster: Kubernetes | Namespace: default | Verified via browser at port 30008_

<figure><img src=".gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
