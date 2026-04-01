# Kubernetes Shared Volumes

We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario. Below you can find more details about it.

1. Create a pod named `volume-share-datacenter`.
2. For the first container, use image `fedora` with `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-datacenter-1`, and run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/media`.
3. For the second container, use image `fedora` with the `latest` tag only and remember to mention the tag i.e `fedora:latest`, container should be named as `volume-container-datacenter-2`, and again run a `sleep` command for it so that it remains in running state. Volume `volume-share` should be mounted at path `/tmp/cluster`.
4. Volume name should be `volume-share` of type `emptyDir`.
5. After creating the pod, exec into the first container i.e `volume-container-datacenter-1`, and just for testing create a file `media.txt` with the content `Welcome to xFusionCorp Industries` under the mounted path of first container i.e `/tmp/media`.
6. The file `media.txt` should be present under the mounted path `/tmp/cluster` on the second container `volume-container-datacenter-2` as well, since they are using a shared volume.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.





***

### 1. Create the Pod with shared volume

Create a YAML file:

```bash
vi volume-share.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-datacenter
spec:
  containers:
  - name: volume-container-datacenter-1
    image: fedora:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/media

  - name: volume-container-datacenter-2
    image: fedora:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/cluster

  volumes:
  - name: volume-share
    emptyDir: {}
```

***

### 2. Apply the Pod

```bash
kubectl apply -f volume-share.yaml
```

***

### 3. Verify Pod is running

```bash
kubectl get pods
```

***

### 4. Exec into first container and create file

```bash
kubectl exec -it volume-share-datacenter -c volume-container-datacenter-1 -- /bin/sh
```

Inside the container:

```bash
echo "Welcome to xFusionCorp Industries" > /tmp/media/media.txt
exit
```

***

### 5. Verify file in second container

```bash
kubectl exec -it volume-share-datacenter -c volume-container-datacenter-2 -- /bin/sh
```

Inside the container:

```bash
cat /tmp/cluster/media.txt
```

Expected output:

```
Welcome to xFusionCorp Industries
```

***

### ✅ Summary

* Created a pod with **two containers**
* Used **emptyDir shared volume**
* Mounted at:
  * `/tmp/media` (container 1)
  * `/tmp/cluster` (container 2)
* Verified file sharing across containers

***



```
thor@jump-host ~$ vi volume-share.yaml
thor@jump-host ~$ kubectl apply -f volume-share.yaml
pod/volume-share-datacenter created
thor@jump-host ~$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
volume-share-datacenter   2/2     Running   0          6s
thor@jump-host ~$ kubectl exec -it volume-share-datacenter -c volume-container-datacenter-1 -- /bin/sh
sh-5.3# echo "Welcome to xFusionCorp Industries" > /tmp/media/media.txt
sh-5.3# exit
exit
thor@jump-host ~$ kubectl exec -it volume-share-datacenter -c volume-container-datacenter-2 -- /bin/sh
sh-5.3# cat /tmp/cluster/media.txt
Welcome to xFusionCorp Industries
sh-5.3# exit
exit
thor@jump-host ~$ cat volume-share.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-datacenter
spec:
  containers:
  - name: volume-container-datacenter-1
    image: fedora:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/media

  - name: volume-container-datacenter-2
    image: fedora:latest
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/cluster

  volumes:
  - name: volume-share
    emptyDir: {}
thor@jump-host ~$ 
```
