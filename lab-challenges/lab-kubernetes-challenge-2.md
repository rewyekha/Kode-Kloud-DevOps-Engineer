# Lab- Kubernetes Challenge 2

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>



## Kubernetes Cluster Troubleshooting and Image Gallery Deployment

### Overview

This lab involves diagnosing and repairing a broken two-node Kubernetes cluster, then deploying a file server application that serves an image gallery. The exercise covers API server recovery, kubeconfig correction, node scheduling, CoreDNS image management, persistent storage provisioning, and service exposure.

***

### Prerequisites

* SSH access to the control plane node
* Access to `/etc/kubernetes/manifests/` for static pod configuration
* The `crictl` utility available for container runtime inspection
* Familiarity with `kubectl`, `openssl`, and basic Linux commands

***

### Architecture Summary

The deployment consists of the following components in the `default` namespace:

* A `PersistentVolume` named `data-pv` backed by a hostPath at `/web` on node01
* A `PersistentVolumeClaim` named `data-pvc` bound to `data-pv`
* A `Pod` named `gop-file-server` running the `kodekloud/fileserver` image
* A `NodePort` Service named `gop-fs-service` exposing port 8080 on NodePort 31200
* Static image files copied from `/media` on the control plane to `/web` on node01

***

### Part 1: Cluster Troubleshooting

#### Issue 1: kube-apiserver Not Responding

**Symptom:** All `kubectl` commands fail with a connection refused error targeting port 6433 instead of the correct port 6443.

```
The connection to the server controlplane:6433 was refused - did you specify the right host or port?
```

**Diagnosis:** Inspect the kubeconfig file to identify the misconfigured server port.

```bash
cat /root/.kube/config
```

The `server` field showed `https://controlplane:6433`, which is incorrect. The API server manifest confirmed the correct port:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep secure-port
```

Output confirms `--secure-port=6443`.

**Fix:** Edit the kubeconfig and correct the port from `6433` to `6443`.

```bash
vi /root/.kube/config
```

Change:

```yaml
server: https://controlplane:6433
```

To:

```yaml
server: https://controlplane:6443
```

**Verification:** After correcting the port, run:

```bash
kubectl get nodes
```

If the API server is still not responding, proceed to inspect the kube-apiserver static pod.

***

#### Issue 2: kube-apiserver Static Pod Misconfiguration

**Symptom:** After correcting the kubeconfig port, the API server remains unreachable. The container shows as `Exited` in `crictl`.

**Diagnosis:**

```bash
crictl ps -a | grep kube-apiserver
```

The container was in `Exited` state. Reviewing the static pod manifest revealed the following misconfiguration:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep client-ca-file
```

The flag pointed to a non-existent file:

```
--client-ca-file=/etc/kubernetes/pki/ca-authority.crt
```

Listing the PKI directory confirmed that this file does not exist:

```bash
ls /etc/kubernetes/pki/
```

The correct file is `ca.crt`.

**Fix:** Edit the kube-apiserver manifest and correct the CA file path.

```bash
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Change:

```yaml
- --client-ca-file=/etc/kubernetes/pki/ca-authority.crt
```

To:

```yaml
- --client-ca-file=/etc/kubernetes/pki/ca.crt
```

Saving the file triggers the kubelet to restart the static pod automatically.

**Verification:**

```bash
crictl ps | grep kube-apiserver
```

The container should now show `Running`. Then confirm cluster access:

```bash
kubectl get nodes
```

***

#### Issue 3: node01 in SchedulingDisabled State

**Symptom:** After the API server is restored, `kubectl get nodes` shows node01 with status `Ready,SchedulingDisabled`, indicating it was cordoned.

**Fix:**

```bash
kubectl uncordon node01
```

**Verification:**

```bash
kubectl get nodes
```

Both nodes should now show `Ready` with no scheduling restrictions.

***

#### Issue 4: CoreDNS Running Incorrect Image

**Symptom:** The CoreDNS deployment was using an outdated or incorrect image (`registry.k8s.io/kubedns:1.3.1` visible in the last-applied annotation). The required image is `registry.k8s.io/coredns/coredns:v1.8.6`.

**Fix:**

```bash
kubectl -n kube-system set image deployment/coredns \
  coredns=registry.k8s.io/coredns/coredns:v1.8.6
```

**Verification:**

```bash
kubectl -n kube-system get pods -l k8s-app=kube-dns
kubectl -n kube-system get deploy coredns -o yaml | grep image
```

Both CoreDNS pods should reach `Running` status and the image field should reflect `v1.8.6`.

***

### Part 2: Storage Provisioning

#### Create the PersistentVolume

The PV uses a `hostPath` pointing to `/web` on node01, with `ReadWriteMany` access and 1Gi capacity.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: data-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /web
```

Apply with:

```bash
kubectl apply -f pv.yaml
```

#### Create the PersistentVolumeClaim

The PVC binds directly to `data-pv` by specifying `volumeName`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  volumeName: data-pv
```

Apply with:

```bash
kubectl apply -f pvc.yaml
```

**Verification:**

```bash
kubectl get pv,pvc
```

The PV should show `STATUS: Bound` and the PVC should show `VOLUME: data-pv`.

***

### Part 3: Copy Image Files to node01

Create the `/web` directory on node01 and copy all image files from the control plane's `/media` directory.

```bash
ssh node01 "mkdir -p /web"

scp -r /media/* node01:/web/
```

**Verification:**

```bash
ssh node01 "ls /web"
```

Expected output:

```
kodekloud-cka.png
kodekloud-ckad.png
kodekloud-cks.png
```

***

### Part 4: Deploy the File Server Pod

The `gop-file-server` pod runs the `kodekloud/fileserver` image and mounts the PVC at `/web` to serve the image files.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gop-file-server
  labels:
    run: gop-file-server
spec:
  volumes:
    - name: data-store
      persistentVolumeClaim:
        claimName: data-pvc

  containers:
    - name: gop-file-server
      image: kodekloud/fileserver
      ports:
        - containerPort: 8080
      volumeMounts:
        - mountPath: /web
          name: data-store
```

Apply with:

```bash
kubectl apply -f pod.yaml
```

**Verification:**

```bash
kubectl get pods -o wide
```

The pod should be `Running` and scheduled on node01, since that is where the `/web` hostPath data resides.

***

### Part 5: Create the NodePort Service

Expose the file server pod via a NodePort service on port 31200.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: gop-fs-service
spec:
  type: NodePort
  selector:
    run: gop-file-server
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 31200
```

Apply with:

```bash
kubectl apply -f service.yaml
```

**Verification:**

```bash
kubectl get svc gop-fs-service
```

Expected output:

```
NAME             TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
gop-fs-service   NodePort   172.20.245.136   <none>        8080:31200/TCP   ...
```

The image gallery is now accessible at `http://<node-ip>:31200`.

***

### Key Observations and Troubleshooting

**Two separate API server issues:** The cluster had both a kubeconfig port mismatch (`6433` instead of `6443`) and an incorrect CA certificate path in the API server manifest (`ca-authority.crt` instead of `ca.crt`). Both must be resolved for the API server to become healthy. Fixing only the kubeconfig is insufficient if the static pod itself is misconfigured.

**Static pod restart behavior:** Editing a file under `/etc/kubernetes/manifests/` triggers an automatic restart of the associated static pod by the kubelet. There is no need to manually restart any service. Monitor recovery with `crictl ps | grep kube-apiserver`.

**CoreDNS edit failure:** Attempts to use `kubectl edit` on the CoreDNS deployment failed due to a validation error caused by stale annotations containing the old image reference. Using `kubectl set image` bypasses this issue and updates the image directly without triggering annotation validation conflicts.

**containerPort declaration:** The initial pod definition omitted the `containerPort` field. While this does not prevent connectivity, it is required for certain lab validation checks. The pod was deleted and recreated with the port explicitly declared.

**Service type change:** The initial service was created as `ClusterIP`. It was deleted and recreated as `NodePort` with `nodePort: 31200` to allow external access to the image gallery.

**hostPath and pod scheduling:** Because the PV uses a `hostPath` on node01, the file server pod must run on node01 to access the data. In this lab the scheduler placed it there correctly. In production environments, `nodeAffinity` or `nodeSelector` rules should be used to enforce this placement explicitly.

***

### Resource Summary

| Resource              | Name            | Namespace      | Notes                         |
| --------------------- | --------------- | -------------- | ----------------------------- |
| PersistentVolume      | data-pv         | cluster-scoped | hostPath: /web, ReadWriteMany |
| PersistentVolumeClaim | data-pvc        | default        | Bound to data-pv              |
| Pod                   | gop-file-server | default        | Scheduled on node01           |
| Service               | gop-fs-service  | default        | NodePort 31200                |

### Fixes Applied Summary

| Issue                              | Root Cause                                                | Resolution                                                      |
| ---------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------- |
| API server unreachable             | kubeconfig server port set to 6433                        | Corrected to 6443 in /root/.kube/config                         |
| kube-apiserver pod in Exited state | --client-ca-file pointed to non-existent ca-authority.crt | Corrected to ca.crt in kube-apiserver.yaml                      |
| node01 not scheduling pods         | Node was cordoned                                         | Ran kubectl uncordon node01                                     |
| CoreDNS using wrong image          | Deployment had outdated image reference                   | Updated to registry.k8s.io/coredns/coredns:v1.8.6 via set image |

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>
