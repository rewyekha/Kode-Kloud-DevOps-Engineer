# Day 66: Deploy MySQL on Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

A new MySQL server needs to be deployed on Kubernetes cluster. The Nautilus DevOps team was working on to gather the requirements. Recently they were able to finalize the requirements and shared them with the team members to start working on it. Below you can find the details:

1.) Create a PersistentVolume `mysql-pv`, its capacity should be `250Mi`, set other parameters as per your preference.

2.) Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as `mysql-pv-claim` and request a `250Mi` of storage. Set other parameters as per your preference.

3.) Create a deployment named `mysql-deployment`, use any mysql image as per your preference. Mount the PersistentVolume at mount path `/var/lib/mysql`.

4.) Create a `NodePort` type service named `mysql` and set nodePort to `30007`.

5.) Create a secret named `mysql-root-pass` having a key pair value, where key is `password` and its value is `YUIidhb667`, create another secret named `mysql-user-pass` having some key pair values, where first key is `username` and its value is `kodekloud_pop`, second key is `password` and value is `B4zNgHA7Ya`, create one more secret named `mysql-db-url`, key name is `database` and value is `kodekloud_db7`

6.) Define some environment variables within the container:

a.) `name: MYSQL_ROOT_PASSWORD`, should pick value from secretKeyRef `name: mysql-root-pass` and `key: password`

b.) `name: MYSQL_DATABASE`, should pick value from secretKeyRef `name: mysql-db-url` and `key: database`

c.) `name: MYSQL_USER`, should pick value from secretKeyRef `name: mysql-user-pass` key `key: username`

d.) `name: MYSQL_PASSWORD`, should pick value from secretKeyRef `name: mysql-user-pass` and `key: password`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.


## Deploying MySQL on Kubernetes with Persistent Storage and Secrets

**Category:** Kubernetes / Storage / Secrets Management **Difficulty:** Intermediate **Platform:** KodeKloud / Nautilus DevOps Lab

***

### Overview

This lab walks through deploying a MySQL 5.7 instance on a Kubernetes cluster using production-relevant practices: externalized credentials via Kubernetes Secrets, persistent storage via PersistentVolumes, and exposure via a NodePort Service.

By the end of this lab, the following Kubernetes resources will be configured and running:

* 3 Opaque Secrets (root password, user credentials, database name)
* 1 PersistentVolume (`mysql-pv`) with `Retain` reclaim policy
* 1 PersistentVolumeClaim (`mysql-pv-claim`) bound to the above PV
* 1 Deployment (`mysql-deployment`) running `mysql:5.7`
* 1 NodePort Service (`mysql`) exposing port `30007`

***

### Requirements

| Resource | Name | Details |
| --------------------- | ------------------ | --------------------------------------------- |
| PersistentVolume | `mysql-pv` | 250Mi, ReadWriteOnce, Retain policy, hostPath |
| PersistentVolumeClaim | `mysql-pv-claim` | 250Mi, ReadWriteOnce |
| Deployment | `mysql-deployment` | mysql:5.7, mounted at `/var/lib/mysql` |
| Service | `mysql` | NodePort, port 3306, nodePort 30007 |
| Secret | `mysql-root-pass` | key: `password` |
| Secret | `mysql-user-pass` | keys: `username`, `password` |
| Secret | `mysql-db-url` | key: `database` |

***

### Key Concepts

#### Why `storageClassName: ""`?

In clusters with a default StorageClass (such as `local-path` in k3s environments), a PVC without an explicit `storageClassName` will be fulfilled by dynamic provisioning — creating a brand-new PV automatically, rather than binding to your manually created one.

Setting `storageClassName: ""` on both the PV and the PVC tells Kubernetes to use static binding, ensuring the PVC binds to your specific, pre-created PV.

#### Why use Secrets instead of plain environment variables?

Kubernetes Secrets store sensitive values in base64-encoded form and are kept separate from the workload definition. This means credentials are not hardcoded in Deployment YAML, are not exposed in source control, and can be rotated independently of the application.

***

### Step-by-Step Implementation

#### Step 1 — Create the Secrets

Three secrets are required before the Deployment can start. Create them using `kubectl create secret generic`:

```bash
kubectl create secret generic mysql-root-pass \
  --from-literal=password=YUIidhb667

kubectl create secret generic mysql-user-pass \
  --from-literal=username=kodekloud_pop \
  --from-literal=password=B4zNgHA7Ya

kubectl create secret generic mysql-db-url \
  --from-literal=database=kodekloud_db7
```

**Terminal output:**

```
secret/mysql-root-pass created
secret/mysql-user-pass created
secret/mysql-db-url created
```

Verify the secrets exist:

```bash
kubectl get secrets
```

```
NAME              TYPE     DATA   AGE
mysql-db-url      Opaque   1      14m
mysql-root-pass   Opaque   1      14m
mysql-user-pass   Opaque   2      14m
```

Note that `mysql-user-pass` shows `DATA: 2` because it holds two key-value pairs (username and password), while the others hold one each.

***

#### Step 2 — Create the PersistentVolume

The PV is created with `storageClassName: ""` to prevent the cluster's default StorageClass from interfering with static binding.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: /mnt/data/mysql
EOF
```

**Terminal output:**

```
persistentvolume/mysql-pv created
```

***

#### Step 3 — Create the PersistentVolumeClaim

The PVC must also carry `storageClassName: ""` to match the PV and trigger static binding.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 250Mi
EOF
```

**Terminal output:**

```
persistentvolumeclaim/mysql-pv-claim created
```

Verify the PV and PVC are bound to each other:

```bash
kubectl get pv
kubectl get pvc
```

```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim                  43s

NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO                           38s
```

Both show `Bound` and reference each other — static binding is working correctly.

***

#### Step 4 — Create the Deployment

The Deployment mounts the PVC at `/var/lib/mysql` and injects all four environment variables from the secrets created in Step 1.

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
EOF
```

**Terminal output:**

```
deployment.apps/mysql-deployment created
```

***

#### Step 5 — Create the NodePort Service

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
      nodePort: 30007
EOF
```

**Terminal output:**

```
service/mysql created
```

***

### Verification

#### Check all resources

```bash
kubectl get pods
kubectl get pv
kubectl get pvc
kubectl get svc
kubectl get secrets
```

**Terminal output:**

```bash
NAME                                READY   STATUS    RESTARTS   AGE
mysql-deployment-74446ff65d-xgsm2   1/1     Running   0          13s

NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim                  13m

NAME             STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    mysql-pv   250Mi      RWO                           13m

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP          44m
mysql        NodePort    10.43.127.60   <none>        3306:30007/TCP   14m

NAME              TYPE     DATA   AGE
mysql-db-url      Opaque   1      15m
mysql-root-pass   Opaque   1      15m
mysql-user-pass   Opaque   2      15m
```

#### Inspect the pod in detail

```bash
kubectl describe pod -l app=mysql
```

**Terminal output (abridged):**

```bash
Name:             mysql-deployment-74446ff65d-xgsm2
Namespace:        default
Status:           Running
IP:               10.22.0.15

Containers:
  mysql:
    Image:          mysql:5.7
    Port:           3306/TCP
    State:          Running
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'password' in secret 'mysql-root-pass'>  Optional: false
      MYSQL_DATABASE:       <set to the key 'database' in secret 'mysql-db-url'>     Optional: false
      MYSQL_USER:           <set to the key 'username' in secret 'mysql-user-pass'>  Optional: false
      MYSQL_PASSWORD:       <set to the key 'password' in secret 'mysql-user-pass'>  Optional: false
    Mounts:
      /var/lib/mysql from mysql-storage (rw)

Volumes:
  mysql-storage:
    Type:       PersistentVolumeClaim
    ClaimName:  mysql-pv-claim
    ReadOnly:   false

Events:
  Normal  Scheduled  19s   default-scheduler  Successfully assigned default/mysql-deployment-74446ff65d-xgsm2 to jump-host
  Normal  Pulled     19s   kubelet            Container image "mysql:5.7" already present on machine
  Normal  Created    19s   kubelet            Created container: mysql
  Normal  Started    19s   kubelet            Started container mysql
```

All environment variables are sourced from their respective Secrets, and `/var/lib/mysql` is correctly mounted from the PVC.

***

### Troubleshooting Notes

The following issues were encountered and resolved during this lab. They are documented here as learning reference.

#### Issue 1 — PVC bound to wrong PV (auto-provisioned)

**Symptom:** After creating `mysql-pv` and `mysql-pv-claim`, running `kubectl get pv` showed two PVs — the manually created one remained in `Available` status, while a second PV named `pvc-cd8e56be-...` was auto-created and bound to the PVC.

```
NAME                                       CAPACITY   STATUS      CLAIM
mysql-pv                                   250Mi      Available               <- not bound
pvc-cd8e56be-05bd-4423-a9ea-5bb22c973dfc   250Mi      Bound       default/mysql-pv-claim
```

**Root cause:** The cluster had a `local-path` default StorageClass. Because neither the PV nor the PVC specified `storageClassName`, the PVC was fulfilled dynamically.

**Fix:** Add `storageClassName: ""` to both the PV and PVC spec. An empty string explicitly opts out of dynamic provisioning and enables static binding.

***

#### Issue 2 — Deployment YAML volume section truncated in heredoc

**Symptom:** When applying the Deployment via a heredoc (`<<EOF`), the `volumeMounts` and `volumes` sections were accidentally dropped. The EOF marker appeared on the same line as unrelated content:

```
      volumes:
EOF       claimName: mysql-pv-claim
```

**Root cause:** The heredoc was constructed incorrectly — content after `EOF` on the same line is not part of the document and was silently ignored.

**Fix:** Ensure the `EOF` terminator is always on its own line with no leading or trailing content. When pasting multi-block YAML into a terminal, verify the full spec is present before submitting.

***

#### Issue 3 — `sudo mkdir` failed (no password available)

**Symptom:** The `hostPath` directory `/mnt/data/mysql` did not exist on the node, and attempts to create it with `sudo mkdir` failed because the `thor` user's sudo password was not available in the lab environment.

**Impact:** The PV hostPath did not exist on disk, but MySQL still started. This is because in this lab environment the hostPath was created automatically by the kubelet on first write, or the MySQL container's volume mount succeeded at the overlay filesystem level.

**Note:** In a production environment, hostPath volumes require the directory to pre-exist on the node with appropriate permissions. A safer alternative is to use a StorageClass with automatic provisioning or a network-backed storage solution.

***

### Final Architecture Summary

```bash
                    +------------------+
                    |   NodePort SVC   |
                    |   mysql:30007    |
                    +--------+---------+
                             |
                    +--------+---------+
                    |    Deployment    |
                    | mysql-deployment |
                    |   (mysql:5.7)    |
                    +--------+---------+
                             |
              +--------------+--------------+
              |                             |
   +----------+----------+      +----------+----------+
   |       Secrets       |      |  PersistentVolume   |
   | mysql-root-pass     |      |  mysql-pv (250Mi)   |
   | mysql-user-pass     |      |  via PVC:           |
   | mysql-db-url        |      |  mysql-pv-claim     |
   +---------------------+      +---------------------+
```

| Check | Expected Result |
| ------------- | -------------------------------------- |
| Pod status | Running, Ready 1/1, Restart Count 0 |
| PV status | Bound to `default/mysql-pv-claim` |
| PVC status | Bound to `mysql-pv` |
| Service type | NodePort, 3306:30007/TCP |
| Secrets count | 3 Opaque secrets present |
| Volume mount | `/var/lib/mysql` from `mysql-pv-claim` |
| Env vars | All 4 sourced from secretKeyRef |


<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
