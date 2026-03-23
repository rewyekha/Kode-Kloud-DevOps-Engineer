# Deploy ReplicaSet in Kubernetes Cluster

The Nautilus DevOps team is gearing up to deploy applications on a Kubernetes cluster for migration purposes. A team member has been tasked with creating a ReplicaSet outlined below:

1. Create a ReplicaSet using `httpd` image with `latest` tag (ensure to specify as `httpd:latest`) and name it `httpd-replicaset`.
2. Apply labels: `app` as `httpd_app`, `type` as `front-end`.
3. Name the container `httpd-container`. Ensure the replica count is `4`.

`Note:` The `kubectl` utility on `jump_host` is set up to interact with the Kubernetes cluster.



### ✅ Task Breakdown (Requirements Mapping)

| Requirement     | Implementation                    |
| --------------- | --------------------------------- |
| ReplicaSet name | `httpd-replicaset`                |
| Image           | `httpd:latest`                    |
| Replicas        | `4`                               |
| Container name  | `httpd-container`                 |
| Labels          | `app=httpd_app`, `type=front-end` |
| kubectl access  | Already configured on jump\_host  |

***

### 📝 Step 1: Create ReplicaSet YAML

Create a manifest file:

```bash
vi httpd-replicaset.yaml
```

Paste the following content:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: httpd-replicaset
  labels:
    app: httpd_app
    type: front-end
spec:
  replicas: 4
  selector:
    matchLabels:
      app: httpd_app
      type: front-end
  template:
    metadata:
      labels:
        app: httpd_app
        type: front-end
    spec:
      containers:
      - name: httpd-container
        image: httpd:latest
```

***

### 🚀 Step 2: Apply the ReplicaSet

```bash
kubectl apply -f httpd-replicaset.yaml
```

Expected output:

```
replicaset.apps/httpd-replicaset created
```

***

### 🔍 Step 3: Verify ReplicaSet & Pods

#### Check ReplicaSet

```bash
kubectl get rs
```

Expected:

```
httpd-replicaset   4   4   4
```

#### Check Pods

```bash
kubectl get pods --show-labels
```

You should see **4 running pods**, all with labels:

```
app=httpd_app,type=front-end
```

***

### ✅ Final Confirmation

✔ ReplicaSet created successfully\
✔ 4 replicas running\
✔ Correct image and container name\
✔ Labels correctly applied



```
thor@jumphost ~$ vi httpd-replicaset.yaml
thor@jumphost ~$ kubectl apply -f httpd-replicaset.yaml
error: error parsing httpd-replicaset.yaml: error converting YAML to JSON: yaml: line 23: found a tab character that violates indentation
thor@jumphost ~$ vi httpd-replicaset.yaml
thor@jumphost ~$ kubectl apply -f httpd-replicaset.yaml
replicaset.apps/httpd-replicaset created
thor@jumphost ~$ kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
httpd-replicaset   4         4         4       10s
thor@jumphost ~$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
httpd-replicaset-5v4xb   1/1     Running   0          30s   app=httpd_app,type=front-end
httpd-replicaset-lwspd   1/1     Running   0          30s   app=httpd_app,type=front-end
httpd-replicaset-mrq4h   1/1     Running   0          30s   app=httpd_app,type=front-end
httpd-replicaset-r8qpd   1/1     Running   0          30s   app=httpd_app,type=front-end
thor@jumphost ~$ 
```
