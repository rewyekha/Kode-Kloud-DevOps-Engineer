# Set Up Time Check Pod in Kubernetes

The Nautilus DevOps team needs a time check pod created in a specific Kubernetes namespace for logging purposes. Initially, it's for testing, but it may be integrated into an existing cluster later. Here's what's required:

1. Create a pod called `time-check` in the `xfusion` namespace. The pod should contain a container named `time-check`, utilizing the `busybox` image with the `latest` tag (specify as `busybox:latest`).
2. Create a config map named `time-config` with the data `TIME_FREQ=11` in the same namespace.
3. Configure the `time-check` container to execute the command: `while true; do date; sleep $TIME_FREQ;done`. Ensure the result is written `/opt/data/time/time-check.log`. Also, add an environmental variable `TIME_FREQ` in the container, fetching its value from the config map `TIME_FREQ` key.
4. Create a volume `log-volume` and mount it at `/opt/data/time` within the container.

`Note:` The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.

```bash
thor@jumphost ~$ kubectl create namespace xfusion
namespace/xfusion created
thor@jumphost ~$ kubectl create configmap time-config \
> --from-literal=TIME_FREQ=11 \
> n xfusion
error: exactly one NAME is required, got 3
See 'kubectl create configmap -h' for help and examples
thor@jumphost ~$ kubectl create configmap time-config \
  --from-literal=TIME_FREQ=11 \
  -n xfusion
configmap/time-config created
thor@jumphost ~$ vi time-check.yaml
thor@jumphost ~$ kubectl apply -f time-check.yaml
pod/time-check created
thor@jumphost ~$ kubectl get pods -n xfusion
NAME         READY   STATUS    RESTARTS   AGE
time-check   1/1     Running   0          14s
thor@jumphost ~$ kubectl exec -n xfusion time-check -- tail -f /opt/data/time/time-check.log
Wed Jan 14 04:14:09 UTC 2026
Wed Jan 14 04:14:20 UTC 2026
Wed Jan 14 04:14:31 UTC 2026
^C
thor@jumphost ~$ 
```

#### ✅ What this does

* Creates **namespace `xfusion`** (safe even if it already exists)
* Creates a **ConfigMap `time-config`** with `TIME_FREQ=11`
* Creates a **Pod `time-check`**
  * Uses `busybox:latest`
  * Container name: `time-check`
  * Reads `TIME_FREQ` from the ConfigMap
  * Continuously logs date output every `TIME_FREQ` seconds
  * Writes logs to `/opt/data/time/time-check.log`
  * Uses a volume mounted at `/opt/data/time`

***

#### 📄 Kubernetes Manifest

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: xfusion
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: xfusion
data:
  TIME_FREQ: "11"
---
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: xfusion
spec:
  containers:
    - name: time-check
      image: busybox:latest
      command:
        - /bin/sh
        - -c
        - |
          while true; do
            date >> /opt/data/time/time-check.log
            sleep $TIME_FREQ
          done
      env:
        - name: TIME_FREQ
          valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
      volumeMounts:
        - name: log-volume
          mountPath: /opt/data/time
  volumes:
    - name: log-volume
      emptyDir: {}
```

***

#### 🚀 Apply the configuration

```bash
kubectl apply -f time-check.yaml
```

***

#### 🔍 Verification commands (optional)

```bash
kubectl get pods -n xfusion
kubectl exec -n xfusion time-check -- tail -f /opt/data/time/time-check.log
```

***

### **Steps:**

### **Step 1: Create the namespace**

```bash
kubectl create namespace xfusion
```

_(If it already exists, you may see a warning — that’s fine.)_

***

### **Step 2: Create the ConfigMap**

Create a ConfigMap named `time-config` with `TIME_FREQ=11`.

```bash
kubectl create configmap time-config \
  --from-literal=TIME_FREQ=11 \
  -n xfusion
```

***

### **Step 3: Create the Pod YAML file**

Create a file called `time-check.yaml`.

```bash
vi time-check.yaml
```

Paste the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: xfusion
spec:
  containers:
    - name: time-check
      image: busybox:latest
      command:
        - /bin/sh
        - -c
        - |
          while true; do
            date >> /opt/data/time/time-check.log
            sleep $TIME_FREQ
          done
      env:
        - name: TIME_FREQ
          valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
      volumeMounts:
        - name: log-volume
          mountPath: /opt/data/time
  volumes:
    - name: log-volume
      emptyDir: {}
```

Save and exit (`ESC` → `:wq` → `Enter`).

***

### **Step 4: Create the Pod**

```bash
kubectl apply -f time-check.yaml
```

***

### **Step 5: Verify the Pod**

Check that the pod is running:

```bash
kubectl get pods -n xfusion
```

***

### **Step 6: Verify logging inside the Pod**

Check that the log file is being written:

```bash
kubectl exec -n xfusion time-check -- tail -f /opt/data/time/time-check.log
```

You should see timestamps printed every **11 seconds**.

***

### ✅ **Summary**

✔ Namespace created\
✔ ConfigMap created\
✔ Pod running with BusyBox\
✔ Environment variable sourced from ConfigMap\
✔ Volume mounted at `/opt/data/time`\
✔ Logs written to `time-check.log`
