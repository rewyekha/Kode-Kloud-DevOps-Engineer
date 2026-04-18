# Day 62: Manage Secrets in Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The Nautilus DevOps team is working to deploy some tools in Kubernetes cluster. Some of the tools are licence based so that licence information needs to be stored securely within Kubernetes cluster. Therefore, the team wants to utilize Kubernetes secrets to store those secrets. Below you can find more details about the requirements:

1. We already have a secret key file `blog.txt` under the `/opt/` directory. Create a `generic secret` named `blog`, it should contain the password/license-number present in `blog.txt` file.
2. Also create a `pod` named `secret-nautilus`.
3. Configure pod's `spec` as container name should be `secret-container-nautilus`, image should be `fedora` with `latest` tag (remember to mention the tag with image). Use `sleep` command for container so that it remains in running state. Consume the created secret and mount it under `/opt/apps` within the container.
4. To verify you can exec into the container `secret-container-nautilus`, to check the secret key under the mounted path `/opt/apps`. Before hitting the `Check` button please make sure pod/pods are in running state, also validation can take some time to complete so keep patience.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.


***

## Create Pod YAML

Create a pod definition file:

```bash
vi secret-pod.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-nautilus
spec:
  containers:
    - name: secret-container-nautilus
      image: fedora:latest
      command: ["/bin/sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: secret-volume
          mountPath: /opt/apps
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: blog
```

```bash
thor@jump-host ~$ kubectl create secret generic blog --from-file=/opt/blog.txt
secret/blog created
thor@jump-host ~$ ls /opt/
blog.txt    cni         containerd
thor@jump-host ~$ vi secret-pod.yaml
thor@jump-host ~$ kubectl apply -f secret-pod.yml
error: the path "secret-pod.yml" does not exist
thor@jump-host ~$ kubectl apply -f secret-pod.yaml
pod/secret-nautilus created
thor@jump-host ~$ kubectl get pods
NAME              READY   STATUS    RESTARTS   AGE
secret-nautilus   1/1     Running   0          10s
thor@jump-host ~$ kubectl exec -it secret-nautilus -- /bin/sh
sh-5.3# ls /opt/apps
blog.txt
sh-5.3# cat /opt/apps/blog.txt
5ecur3
sh-5.3# exit
exit
thor@jump-host ~$ cat /opt/blog.txt
5ecur3
thor@jump-host ~$ 

```

<figure><img src=".gitbook/assets/Screenshot 2026-03-27 074745.png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
