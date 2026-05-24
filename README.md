# Lab- Kubernetes Challenge 1

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>



## Deploying a Jekyll Static Site Generator on Kubernetes

### Overview

This lab covers the end-to-end deployment of a Jekyll Static Site Generator (SSG) on a Kubernetes cluster. The exercise demonstrates the configuration of RBAC (Role-Based Access Control), persistent storage, init containers, and NodePort services within a dedicated namespace.

***

### Prerequisites

* A running Kubernetes cluster with `kubectl` configured
* Access to the Kubernetes CA certificate and key at `/etc/kubernetes/pki/`
* The `openssl` utility installed on the control plane node

***

### Architecture Summary

The deployment consists of the following components within the `development` namespace:

* A `PersistentVolume` named `jekyll-site` using local storage
* A `PersistentVolumeClaim` named `jekyll-site` bound to the above PV
* A `Pod` named `jekyll` with an init container to scaffold the Jekyll site
* A `NodePort` Service named `jekyll-node-service` exposing port 4000
* A `Role` named `developer-role` with full permissions on pods, services, and PVCs
* A `RoleBinding` named `developer-rolebinding` associating the role with user `martin`
* A kubeconfig user entry and context for `martin` to interact with the cluster

***

### Step 1: Generate TLS Credentials for User martin

Create a private key and a certificate signing request (CSR) for the user `martin`, then sign the certificate using the cluster CA.

```bash
# Generate a private key and CSR
openssl genrsa -out martin.key 2048
openssl req -new -key martin.key -subj "/CN=martin" -out martin.csr

# Sign the CSR with the cluster CA
openssl x509 -req \
  -in martin.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -out martin.crt \
  -CAcreateserial
```

The resulting files `martin.key` and `martin.crt` are used in subsequent kubeconfig steps.

***

### Step 2: Configure Kubeconfig for User martin

Add the user credentials and create a new context called `developer` in the default kubeconfig file.

```bash
# Configure kubeconfig for Martin
kubectl config set-credentials martin \
  --client-certificate=martin.crt \
  --client-key=martin.key

kubectl config set-context developer \
  --cluster=kubernetes \
  --user=martin
```

To verify the context was created:

```bash
kubectl config get-contexts
```

***

### Step 3: Create RBAC Resources

#### Role

Create `developer-role` in the `development` namespace with full permissions on pods, services, and persistent volume claims.

```bash
kubectl create role developer-role \
  --resource=pods,services,persistentvolumeclaims \
  --verb='*' \
  -n development
```

#### RoleBinding

Bind the role to user `martin`.

```bash
kubectl create rolebinding developer-rolebinding \
  --role=developer-role \
  --user=martin \
  -n development
```

***

### Step 4: Create the PersistentVolumeClaim

The `jekyll-site` PVC binds to the pre-existing `jekyll-site` PersistentVolume, which has `ReadWriteMany` access mode and uses the `local-storage` storage class.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jekyll-site
  namespace: development
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```

Apply with:

```bash
kubectl apply -f pvc.yaml
```

Confirm binding:

```bash
kubectl get pvc -n development
```

Expected output shows `STATUS: Bound` and `VOLUME: jekyll-site`.

***

### Step 5: Deploy the Jekyll Pod

The pod uses an init container to scaffold a new Jekyll site into the shared volume before the main container begins serving.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: jekyll
  namespace: development
  labels:
    run: jekyll
spec:
  volumes:
    - name: site
      persistentVolumeClaim:
        claimName: jekyll-site

  initContainers:
    - name: copy-jekyll-site
      image: gcr.io/kodekloud/customimage/jekyll
      command:
        - /bin/sh
        - -c
        - "rm -rf /site/* && jekyll new /site && cd /site && bundle install"
      volumeMounts:
        - name: site
          mountPath: /site

  containers:
    - name: jekyll
      image: gcr.io/kodekloud/customimage/jekyll-serve
      ports:
        - containerPort: 4000
      command:
        - /bin/sh
        - -c
        - "cd /site && bundle install && bundle exec jekyll serve --host 0.0.0.0 --port 4000"
      volumeMounts:
        - name: site
          mountPath: /site
```

Apply with:

```bash
kubectl apply -f pod.yaml
```

Monitor init container progress:

```bash
kubectl get pods -n development -w
```

The pod will transition from `Init:0/1` to `Running` once the init container completes.

***

### Step 6: Create the NodePort Service

Expose the Jekyll pod via a NodePort service on port 30097.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: jekyll-node-service
  namespace: development
spec:
  type: NodePort
  selector:
    run: jekyll
  ports:
    - port: 4000
      targetPort: 4000
      nodePort: 30097
```

Apply with:

```bash
kubectl apply -f service.yaml
```

Verify the service:

```bash
kubectl get svc -n development
```

Expected output:

```
NAME                  TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
jekyll-node-service   NodePort   172.20.89.139    <none>        4000:30097/TCP   ...
```

***

### Step 7: Set the developer Context as Current

Switch the active kubeconfig context to `developer` for subsequent operations as user `martin`.

```bash
kubectl config use-context developer
```

***

### Step 8: Validate the Deployment

Confirm the Jekyll site is accessible via the node's IP on the NodePort:

```bash
curl http://<node-ip>:30097
```

A successful response returns the Jekyll default site HTML, including the page title and a welcome post dated to the current date.

To return to the admin context at any time:

```bash
kubectl config use-context kubernetes-admin@kubernetes
```

***

### Key Observations and Troubleshooting

**Initial service misconfiguration:** The first service definition omitted the `port` and `targetPort` fields and had a formatting error in the `nodePort` value. The service was deleted and recreated with the correct port mapping (`port: 4000`, `targetPort: 4000`, `nodePort: 30097`). Always verify that all three port fields are explicitly defined for NodePort services.

**RBAC scope:** The `developer-role` grants permissions only on pods, services, and persistent volume claims. Running `kubectl get all` as user `martin` will produce Forbidden errors for other resource types such as deployments, daemonsets, and stateful sets. This is expected behavior and does not indicate a misconfiguration.

**Init container dependency:** The main `jekyll` container depends on the init container completing successfully. If the pod remains in `Init:0/1` for an extended period, inspect the init container logs:

```bash
kubectl logs jekyll -n development -c copy-jekyll-site
```

**PVC binding:** The PVC will only bind if the PV's access mode, storage class, and capacity are compatible. Confirm the PV exists and is in `Available` status before applying the PVC:

```bash
kubectl get pv
```

***

### Resource Summary

| Resource              | Name                  | Namespace          |
| --------------------- | --------------------- | ------------------ |
| PersistentVolume      | jekyll-site           | cluster-scoped     |
| PersistentVolumeClaim | jekyll-site           | development        |
| Pod                   | jekyll                | development        |
| Service               | jekyll-node-service   | development        |
| Role                  | developer-role        | development        |
| RoleBinding           | developer-rolebinding | development        |
| Kubeconfig User       | martin                | default kubeconfig |
| Kubeconfig Context    | developer             | default kubeconfig |

<figure><img src=".gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
