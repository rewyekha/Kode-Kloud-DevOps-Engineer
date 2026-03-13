# Weight: 9

The Nautilus application development team aims to test a straightforward deployment by creating an Nginx-based Pod on the Kubernetes cluster. The specifications for this deployment are as follows:

Create a Pod named `dummy-nginx-nginx-t1q6`, it must use `nginx:stable-alpine3.17-slim` image. Finally, ensure the Pod remains in the `Running` state.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.

Pod 'dummy-nginx-nginx-t1q6' exists

Pod is in 'Running' state

Pod is using image 'nginx:stable-alpine3.17-slim'



#### Option 1 — Using YAML (recommended)

Create the manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dummy-nginx-nginx-t1q6
spec:
  containers:
    - name: nginx
      image: nginx:stable-alpine3.17-slim
      ports:
        - containerPort: 80
```

Save it:

```bash
vi dummy-nginx-nginx-t1q6.yaml
```

Apply it:

```bash
kubectl apply -f dummy-nginx-nginx-t1q6.yaml
```

***

#### Option 2 — One-liner command

```bash
kubectl run dummy-nginx-nginx-t1q6 \
  --image=nginx:stable-alpine3.17-slim \
  --restart=Never
```

***

#### Verify

```bash
kubectl get pods
```

You should see:

```
NAME                     READY   STATUS    RESTARTS   AGE
dummy-nginx-nginx-t1q6   1/1     Running   0          ...
```

To confirm the image:

```bash
kubectl describe pod dummy-nginx-nginx-t1q6 | grep Image
```

If the status shows **Running**, then:

✔ Pod exists\
✔ Pod is in Running state\
✔ Pod is using `nginx:stable-alpine3.17-slim`

Done ✅

