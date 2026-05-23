# Lab- Kubernetes Challenge 4

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Deploying a Highly Available Redis Cluster on Kubernetes

### Overview

This lab covers the deployment of a six-node Redis Cluster on Kubernetes using a StatefulSet. The cluster is configured with three master nodes and three replica nodes, providing both horizontal sharding across 16384 hash slots and high availability through automatic failover. Each pod is backed by a dedicated PersistentVolume hosted on the worker node.

***

### Prerequisites

* A running Kubernetes cluster with at least one worker node
* `kubectl` configured with admin access
* SSH access to the worker node for directory creation
* The `redis-cluster-configmap` ConfigMap must already exist in the `default` namespace

***

### Architecture Summary

| Component         | Details                                          |
| ----------------- | ------------------------------------------------ |
| StatefulSet       | redis-cluster, 6 replicas                        |
| Image             | redis:5.0.1-alpine                               |
| Headless Service  | redis-cluster-service (ports 6379, 16379)        |
| PersistentVolumes | redis01 through redis06, 1Gi each, ReadWriteOnce |
| Cluster topology  | 3 masters + 3 replicas, 1 replica per master     |
| Hash slots        | 16384 slots distributed across 3 masters         |

**Cluster topology after initialization:**

* `redis-cluster-0`, `redis-cluster-1`, `redis-cluster-2` are assigned as masters
* `redis-cluster-3`, `redis-cluster-4`, `redis-cluster-5` are assigned as replicas, one for each master

***

### Step 1: Inspect the ConfigMap

The `redis-cluster-configmap` is pre-created in the cluster. Inspect it before proceeding to understand the configuration it provides.

```bash
kubectl get configmap redis-cluster-configmap -o yaml
```

The ConfigMap contains two keys:

**redis.conf** — the Redis server configuration enabling cluster mode:

```
cluster-enabled yes
cluster-require-full-coverage no
cluster-node-timeout 15000
cluster-config-file /data/nodes.conf
cluster-migration-barrier 1
appendonly yes
protected-mode no
```

**update-node.sh** — a shell script that runs before the Redis server starts. It replaces the pod IP in the cluster nodes configuration file to handle pod IP changes across restarts:

```sh
#!/bin/sh
REDIS_NODES="/data/nodes.conf"
sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" ${REDIS_NODES}
exec "$@"
```

The `defaultMode: 0755` setting on the ConfigMap volume mount ensures the script is executable inside the container.

***

### Step 2: Create Directories on the Worker Node

Each PersistentVolume uses a `hostPath` pointing to a directory on the worker node. These directories must exist before the PVs are created.

```bash
ssh node01 "mkdir -p /redis01 /redis02 /redis03 /redis04 /redis05 /redis06"
```

***

### Step 3: Create the PersistentVolumes

Create six PersistentVolumes, one for each Redis pod. Each PV uses `ReadWriteOnce` access mode and a `hostPath` pointing to the directory created in the previous step.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis01
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis02
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis02
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis03
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis03
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis04
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis04
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis05
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis05
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis06
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /redis06
```

Apply:

```bash
kubectl apply -f redis-pvs.yaml
```

Verify all six PVs are in `Available` status:

```bash
kubectl get pv
```

***

### Step 4: Create the Headless Service

A headless service (with `clusterIP: None`) is required for StatefulSet DNS resolution. It exposes the Redis client port and the cluster gossip port used for inter-node communication.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
spec:
  clusterIP: None
  selector:
    app: redis-cluster
  ports:
    - name: client
      port: 6379
      targetPort: 6379
    - name: gossip
      port: 16379
      targetPort: 16379
```

Apply:

```bash
kubectl apply -f redis-service.yaml
```

The headless service enables each pod to be addressed individually via DNS entries of the form `redis-cluster-<ordinal>.redis-cluster-service.default.svc.cluster.local`.

***

### Step 5: Deploy the StatefulSet

The StatefulSet definition includes all required configuration: the update script as the container entrypoint, downward API for pod IP injection, named ports, the ConfigMap volume mount, and a `volumeClaimTemplate` that automatically provisions a PVC for each pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  serviceName: redis-cluster-service
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster

  template:
    metadata:
      labels:
        app: redis-cluster

    spec:
      containers:
        - name: redis
          image: redis:5.0.1-alpine
          command:
            - /conf/update-node.sh
            - redis-server
            - /conf/redis.conf
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: status.podIP
          ports:
            - name: client
              containerPort: 6379
            - name: gossip
              containerPort: 16379
          volumeMounts:
            - name: conf
              mountPath: /conf
              readOnly: false
            - name: data
              mountPath: /data
              readOnly: false

      volumes:
        - name: conf
          configMap:
            name: redis-cluster-configmap
            defaultMode: 0755

  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```

Apply:

```bash
kubectl apply -f redis-statefulset.yaml
```

Monitor pod startup:

```bash
kubectl get pods -w
```

All six pods must reach `Running` status before proceeding. The StatefulSet creates pods sequentially — `redis-cluster-0` through `redis-cluster-5` — and each pod's PVC is automatically bound to one of the available PVs.

***

### Step 6: Initialize the Redis Cluster

Once all six pods are running, initialize the Redis Cluster by connecting to the first pod and running the cluster creation command. The command collects all pod IPs dynamically using a `jsonpath` query.

```bash
kubectl exec -it redis-cluster-0 -- redis-cli --cluster create --cluster-replicas 1 \
  $(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 {end}')
```

When prompted with:

```
Can I set the above configuration? (type 'yes' to accept):
```

Type `yes` and press Enter.

**Expected output after confirmation:**

```
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
>>> Performing Cluster Check (using node 172.17.1.2:6379)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

The `--cluster-replicas 1` flag instructs Redis to assign exactly one replica to each master. With six nodes, the result is three masters and three replicas.

***

### Verification

Check pods, PVCs, and PVs:

```bash
kubectl get pods,pvc,pv
```

All six pods should be `Running`, all six PVCs should be `Bound`, and all six PVs should show `Bound` with the corresponding claim listed.

Verify cluster health from inside a pod:

```bash
kubectl exec -it redis-cluster-0 -- redis-cli cluster info
```

The `cluster_state` field should read `ok` and `cluster_slots_assigned` should be `16384`.

To view the full node topology:

```bash
kubectl exec -it redis-cluster-0 -- redis-cli cluster nodes
```

***

### Notes and Observations

**StatefulSet pod ordering:** The StatefulSet creates pods one at a time in ascending ordinal order and waits for each to become ready before creating the next. This ensures stable, predictable pod names (`redis-cluster-0` through `redis-cluster-5`) which are required for the cluster initialization command to work correctly.

**volumeClaimTemplates and PV binding:** The `volumeClaimTemplate` named `data` causes Kubernetes to create one PVC per pod, named `data-redis-cluster-<ordinal>`. These PVCs bind to the available `redis01` through `redis06` PVs in non-deterministic order. The binding is based on matching access mode and capacity, not name. The final binding can be verified with `kubectl get pvc`.

**Headless service requirement:** StatefulSets require a headless service (`clusterIP: None`) to provide stable DNS hostnames for each pod. The `serviceName` field in the StatefulSet spec must match the name of this headless service exactly.

**update-node.sh and the POD\_IP environment variable:** Each time a Redis pod restarts, it may receive a new IP address. The `update-node.sh` script uses the `POD_IP` environment variable (injected via the downward API) to update the cluster nodes configuration file at `/data/nodes.conf` before Redis starts. Without this script, a restarted pod would advertise its old IP to the cluster and cause communication failures.

**defaultMode: 0755:** The ConfigMap is mounted with `defaultMode: 0755` to ensure that `update-node.sh` has execute permissions. Without this, the container entrypoint will fail with a permission denied error.

**Data persistence across pod restarts:** Because each pod's data directory (`/data`) is backed by a PVC, the cluster state and append-only log survive pod restarts. However, because the underlying PVs use `hostPath`, data is tied to the specific worker node. If node01 is removed or replaced, the data in those hostPath directories would be lost.

***

### Resource Summary

| Resource               | Name                                         | Count | Notes                               |
| ---------------------- | -------------------------------------------- | ----- | ----------------------------------- |
| StatefulSet            | redis-cluster                                | 1     | 6 replicas                          |
| Pods                   | redis-cluster-0 to redis-cluster-5           | 6     | 3 masters, 3 replicas               |
| Headless Service       | redis-cluster-service                        | 1     | Ports 6379 and 16379                |
| PersistentVolumes      | redis01 to redis06                           | 6     | 1Gi each, RWO, hostPath             |
| PersistentVolumeClaims | data-redis-cluster-0 to data-redis-cluster-5 | 6     | Auto-created by volumeClaimTemplate |
| ConfigMap              | redis-cluster-configmap                      | 1     | Pre-existing                        |

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
