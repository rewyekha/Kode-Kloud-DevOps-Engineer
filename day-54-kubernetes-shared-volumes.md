# Day 54: Kubernetes Shared Volumes

We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.

1. Create a pod named `volume-share-devops`.
2. For the first container, use image `debian` with `latest` tag only and remember to mention the tag i.e `debian:latest`, container should be named as `volume-container-devops-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/blog`.
3. For the second container, use image `debian` with the `latest` tag only and remember to mention the tag i.e `debian:latest`, container should be named as `volume-container-devops-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/games`.
4. Volume name should be `volume-share` of type `emptyDir`.
5. After creating the pod, exec into the first container i.e `volume-container-devops-1`, and just for testing create a file `blog.txt` with the content `Welcome to xFusionCorp Industries` under the mounted path of first container i.e `/tmp/blog`.
6. The file `blog.txt` should be present under the mounted path `/tmp/games` on the second container `volume-container-devops-2` as well, since they are using a shared volume.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

## 📘 Kubernetes Shared Volume Between Containers (emptyDir)

### 📌 Objective

Create a Kubernetes Pod with multiple containers that share a common volume using `emptyDir`. Verify that data written in one container is accessible from another.

***

### 🏗️ Pod Requirements

* Pod Name: `volume-share-nautilus`
* Two containers:
  * `volume-container-nautilus-1`
  * `volume-container-nautilus-2`
* Image: `ubuntu:latest`
* Shared Volume:
  * Name: `volume-share`
  * Type: `emptyDir`
* Mount Paths:
  * Container 1 → `/tmp/beta`
  * Container 2 → `/tmp/apps`

***

### 📄 Step 1: Create Pod YAML

```bash
thor@jump-host ~$ vi volume-share-nautilus.yaml
```

#### 🔹 YAML Configuration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
  - name: volume-container-nautilus-1
    image: ubuntu:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/beta

  - name: volume-container-nautilus-2
    image: ubuntu:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/apps

  volumes:
  - name: volume-share
    emptyDir: {}
```

#### 🧠 Explanation

* `emptyDir`: Creates a temporary shared directory for all containers in the pod.
* `volumeMounts`: Mounts the shared volume at different paths inside each container.
* `sleep 3600`: Keeps containers running for testing.

***

### 🚀 Step 2: Apply the Configuration

```bash
thor@jump-host ~$ kubectl apply -f volume-share-nautilus.yaml
```

#### ✅ Output

```bash
pod/volume-share-nautilus created
```

#### 🧠 Explanation

* Creates the pod in the Kubernetes cluster using the YAML definition.

***

### 🔍 Step 3: Verify Pod Status

```bash
thor@jump-host ~$ kubectl get pods
```

#### ✅ Output

```bash
NAME                    READY   STATUS    RESTARTS   AGE
volume-share-nautilus   2/2     Running   0          6s
```

#### 🧠 Explanation

* `2/2`: Both containers are running.
* `Running`: Pod is successfully created and active.

***

### 🔐 Step 4: Access First Container

```bash
thor@jump-host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- /bin/bash
```

#### 🧠 Explanation

* `kubectl exec`: Executes a command inside a container.
* `-c`: Specifies the container name.

***

### ✍️ Step 5: Create File in Shared Volume

```bash
root@volume-share-nautilus:/# echo "Welcome to xFusionCorp Industries" > /tmp/beta/beta.txt
```

#### 🧠 Explanation

* Creates `beta.txt` inside `/tmp/beta` (shared volume).
* Data is written to the `emptyDir` volume.

***

### 🚪 Exit Container

```bash
root@volume-share-nautilus:/# exit
exit
```

***

### 🔐 Step 6: Access Second Container

```bash
thor@jump-host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- /bin/bash
```

***

### 📖 Step 7: Verify Shared File

```bash
root@volume-share-nautilus:/# cat /tmp/apps/beta.txt
```

#### ✅ Output

```bash
Welcome to xFusionCorp Industries
```

#### 🧠 Explanation

* File created in container 1 is visible in container 2.
* Confirms volume is shared correctly.

***

### 🚪 Exit Container

```bash
root@volume-share-nautilus:/# exit
exit
```

***

### 🎯 Final Verification (Full Terminal Session)

```bash
thor@jump-host ~$ vi volume-share-nautilus.yaml
thor@jump-host ~$ kubectl apply -f volume-share-nautilus.yaml
pod/volume-share-nautilus created

thor@jump-host ~$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
volume-share-nautilus   2/2     Running   0          6s

thor@jump-host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- /bin/bash
root@volume-share-nautilus:/# echo "Welcome to xFusionCorp Industries" > /tmp/beta/beta.txt
root@volume-share-nautilus:/# exit
exit

thor@jump-host ~$ kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- /bin/bash
root@volume-share-nautilus:/# cat /tmp/apps/beta.txt
Welcome to xFusionCorp Industries
root@volume-share-nautilus:/# exit
exit
```

***

### 🧠 Key Takeaways

* `emptyDir` volumes:
  * Exist only for the lifetime of the pod
  * Shared across all containers in the pod
* Useful for:
  * Temporary data sharing
  * Inter-container communication

***

### 🏁 Conclusion

This setup demonstrates how multiple containers within the same Pod can communicate and share data efficiently using a shared `emptyDir` volume.
