# Resolve Pod Deployment Issue

A junior DevOps team member encountered difficulties deploying a stack on the Kubernetes cluster. The pod fails to start, presenting errors. Let's troubleshoot and rectify the issue promptly.

1. There is a pod named `webserver`, and the container within it is named `httpd-container`, its utilizing the `httpd:latest` image.
2. Additionally, there's a sidecar container named `sidecar-container` using the `ubuntu:latest` image.

Identify and address the issue to ensure the pod is in the `running` state and the application is accessible.

`Note:` The `kubectl` utility on `jump_host` is configured to interact with the Kubernetes cluster.

```bash
thor@jumphost ~$ kubectl get pods
NAME        READY   STATUS             RESTARTS   AGE
webserver   1/2     ImagePullBackOff   0          88s
thor@jumphost ~$ kubectl describe pod webserver
Name:             webserver
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Thu, 15 Jan 2026 01:08:12 +0000
Labels:           app=web-app
Annotations:      <none>
Status:           Pending
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  nginx-container:
    Container ID:   
    Image:          nginx:latests
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vfvsw (ro)
  sidecar-container:
    Container ID:  containerd://985845da2f0e9e4a2eed8714e3873d103702489f9d1eefb8341de307f439b796
    Image:         ubuntu:latest
    Image ID:      docker.io/library/ubuntu@sha256:c35e29c9450151419d9448b0fd75374fec4fff364a27f176fb458d472dfc9e54
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
    State:          Running
      Started:      Thu, 15 Jan 2026 01:08:16 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vfvsw (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  shared-logs:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-vfvsw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  107s                default-scheduler  Successfully assigned default/webserver to kodekloud-control-plane
  Normal   Pulling    106s                kubelet            Pulling image "ubuntu:latest"
  Normal   Pulled     103s                kubelet            Successfully pulled image "ubuntu:latest" in 2.912180866s (2.912196058s including waiting)
  Normal   Created    103s                kubelet            Created container sidecar-container
  Normal   Started    103s                kubelet            Started container sidecar-container
  Normal   Pulling    60s (x3 over 107s)  kubelet            Pulling image "nginx:latests"
  Warning  Failed     59s (x3 over 106s)  kubelet            Failed to pull image "nginx:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:latests": failed to resolve reference "docker.io/library/nginx:latests": docker.io/library/nginx:latests: not found
  Warning  Failed     59s (x3 over 106s)  kubelet            Error: ErrImagePull
  Normal   BackOff    24s (x6 over 103s)  kubelet            Back-off pulling image "nginx:latests"
  Warning  Failed     24s (x6 over 103s)  kubelet            Error: ImagePullBackOff
thor@jumphost ~$ kubectl get pods webserver -o yaml > pod.yml
thor@jumphost ~$ vi pod.yml
thor@jumphost ~$ kubectl get pods
NAME        READY   STATUS             RESTARTS   AGE
webserver   1/2     ImagePullBackOff   0          3m48s
thor@jumphost ~$ kubectl delete pod webserver
pod "webserver" deleted
c
^Cthor@jumphost ~$ kubect get pods --watch
^Cthor@jumphost ~$ 
thor@jumphost ~$ 
thor@jumphost ~$ kubectl get pods --watch
^Cthor@jumphost ~$ 
thor@jumphost ~$ kubectl get pods --watch

^Cthor@jumphost ~$ 
thor@jumphost ~$ 
thor@jumphost ~$ kubectl get pods 
No resources found in default namespace.
thor@jumphost ~$ ls -l
total 8
-rw-r--r-- 1 thor thor 4301 Jan 15 01:11 pod.yml
thor@jumphost ~$ cat pod.yml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"web-app"},"name":"webserver","namespace":"default"},"spec":{"containers":[{"image":"nginx:latests","name":"nginx-container","volumeMounts":[{"mountPath":"/var/log/nginx","name":"shared-logs"}]},{"command":["sh","-c","while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"],"image":"ubuntu:latest","name":"sidecar-container","volumeMounts":[{"mountPath":"/var/log/nginx","name":"shared-logs"}]}],"volumes":[{"emptyDir":{},"name":"shared-logs"}]}}
  creationTimestamp: "2026-01-15T01:08:12Z"
  labels:
    app: web-app
  name: webserver
  namespace: default
  resourceVersion: "2531"
  uid: 600392fc-fe2b-4df6-aad8-0616b5176769
spec:
  containers:
  - image: nginx:latest
    imagePullPolicy: IfNotPresent
    name: nginx-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/log/nginx
      name: shared-logs
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-vfvsw
      readOnly: true
  - command:
    - sh
    - -c
    - while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep
      30; done
    image: ubuntu:latest
    imagePullPolicy: Always
    name: sidecar-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/log/nginx
      name: shared-logs
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-vfvsw
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: kodekloud-control-plane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: shared-logs
  - name: kube-api-access-vfvsw
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-01-15T01:08:12Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-01-15T01:08:12Z"
    message: 'containers with unready status: [nginx-container]'
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-01-15T01:08:12Z"
    message: 'containers with unready status: [nginx-container]'
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-01-15T01:08:12Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - image: nginx:latests
    imageID: ""
    lastState: {}
    name: nginx-container
    ready: false
    restartCount: 0
    started: false
    state:
      waiting:
        message: Back-off pulling image "nginx:latests"
        reason: ImagePullBackOff
  - containerID: containerd://985845da2f0e9e4a2eed8714e3873d103702489f9d1eefb8341de307f439b796
    image: docker.io/library/ubuntu:latest
    imageID: docker.io/library/ubuntu@sha256:c35e29c9450151419d9448b0fd75374fec4fff364a27f176fb458d472dfc9e54
    lastState: {}
    name: sidecar-container
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-01-15T01:08:16Z"
  hostIP: 172.17.0.2
  phase: Pending
  podIP: 10.244.0.5
  podIPs:
  - ip: 10.244.0.5
  qosClass: BestEffort
  startTime: "2026-01-15T01:08:12Z"
thor@jumphost ~$ kubectl apply -f pod.yml
pod/webserver created
thor@jumphost ~$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          11s
thor@jumphost ~$ kubectl get pods --watch
NAME        READY   STATUS    RESTARTS   AGE
webserver   2/2     Running   0          15s
^Cthor@jumphost ~kubectl describe pod webserver
Name:             webserver
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Thu, 15 Jan 2026 01:14:59 +0000
Labels:           app=web-app
Annotations:      <none>
Status:           Running
IP:               10.244.0.6
IPs:
  IP:  10.244.0.6
Containers:
  nginx-container:
    Container ID:   containerd://dae267b8a7933f3fd870a9df72b90efbf37c7071b0bea0cf91ab2944149566b6
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:c881927c4077710ac4b1da63b83aa163937fb47457950c267d92f7e4dedf4aec
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 15 Jan 2026 01:15:06 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vfvsw (ro)
  sidecar-container:
    Container ID:  containerd://d4e6bc8d98089b70cfb3fbdd951d565351722dd53c52380f2077fde86800cbf5
    Image:         ubuntu:latest
    Image ID:      docker.io/library/ubuntu@sha256:c35e29c9450151419d9448b0fd75374fec4fff364a27f176fb458d472dfc9e54
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
    State:          Running
      Started:      Thu, 15 Jan 2026 01:15:07 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/log/nginx from shared-logs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-vfvsw (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  shared-logs:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-vfvsw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulling  39s   kubelet  Pulling image "nginx:latest"
  Normal  Pulled   33s   kubelet  Successfully pulled image "nginx:latest" in 5.867074338s (5.867092962s including waiting)
  Normal  Created  33s   kubelet  Created container nginx-container
  Normal  Started  33s   kubelet  Started container nginx-container
  Normal  Pulling  33s   kubelet  Pulling image "ubuntu:latest"
  Normal  Pulled   33s   kubelet  Successfully pulled image "ubuntu:latest" in 171.909139ms (171.924853ms including waiting)
  Normal  Created  33s   kubelet  Created container sidecar-container
  Normal  Started  32s   kubelet  Started container sidecar-container
thor@jumphost ~$ 
```

## 🛠️ Fixing `ImagePullBackOff` and Bringing a Multi-Container Pod to Running State

### 📘 Overview

In this task, a Kubernetes pod named **`webserver`** failed to reach the `Running` state due to an image pull error in the main container. The pod also included a sidecar container responsible for log monitoring.

This document explains:

* How to identify the root cause
* How to fix the issue step by step using `kubectl`
* How to validate the final solution

***

### 🧩 Problem Summary

* Pod name: `webserver`
* Containers:
  * **Main container**: `nginx-container`
  * **Sidecar container**: `sidecar-container`
* Pod status: `ImagePullBackOff`
* Website inaccessible

***

### 🔍 Step 1: Identify the Issue

Check the pod status:

```bash
kubectl get pods
```

Output:

```
webserver   1/2   ImagePullBackOff
```

Describe the pod for details:

```bash
kubectl describe pod webserver
```

#### 🔴 Root Cause Identified

From the events section:

```
Failed to pull image "nginx:latests"
```

➡️ The image name is incorrect.\
Correct image should be:

```
nginx:latest
```

***

### 🛠️ Step 2: Export Pod Definition

Export the existing pod YAML so it can be corrected:

```bash
kubectl get pod webserver -o yaml > pod.yml
```

***

### ✏️ Step 3: Fix the Pod Specification

Edit the file:

```bash
vi pod.yml
```

#### ✅ Fix 1: Correct the Image Name

Change:

```yaml
image: nginx:latests
```

To:

```yaml
image: nginx:latest
```

***

#### ✅ Fix 2: Ensure Sidecar Has a Long-Running Process

Confirm the sidecar container has a command like this:

```yaml
command:
- sh
- -c
- while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
```

This ensures the sidecar container does not exit immediately.

***

### 🧹 Step 4: Recreate the Pod

Delete the broken pod:

```bash
kubectl delete pod webserver
```

Apply the corrected configuration:

```bash
kubectl apply -f pod.yml
```

***

### ✅ Step 5: Verify the Fix

Check pod status:

```bash
kubectl get pods
```

Expected output:

```
webserver   2/2   Running
```

Describe the pod to confirm both containers are running:

```bash
kubectl describe pod webserver
```

You should see:

* `nginx-container` → Running
* `sidecar-container` → Running
* Pod conditions: `Ready=True`

***

### 🎯 Final Result

✔ Pod `webserver` is running\
✔ Both containers are healthy\
✔ Correct images are used\
✔ Sidecar container remains alive\
✔ Website is accessible

***

### 🧠 Key Learnings

* `ImagePullBackOff` often means a **typo in the image name**
* Always check `kubectl describe pod` for exact error messages
* Sidecar containers **must run a long-lived command**
* Exporting → fixing → reapplying YAML is a safe recovery method

***

### 📌 Quick Exam Tip

> **If a pod is stuck in `ImagePullBackOff`, always verify the image name first.**

***

