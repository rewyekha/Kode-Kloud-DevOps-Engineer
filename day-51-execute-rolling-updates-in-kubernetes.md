# Day 51: Execute Rolling Updates in Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image `nginx:1.19` with the latest updates.

Execute a rolling update for this application, integrating the `nginx:1.19` image. The deployment is named `nginx-deployment`.

Ensure all pods are operational post-update.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Question

An application currently running on the Kubernetes cluster employs the **nginx** web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image **nginx:1.19** with the latest updates.

**Task:** Execute a rolling update for this application, integrating the **nginx:1.19** image. The deployment is named **nginx-deployment**. Ensure all pods are operational post-update.

> **Note:** The `kubectl` utility on the jump-host is already configured to work with the Kubernetes cluster.

***

### Step 1: Check Current Deployment

```bash
thor@jump-host ~$ kubectl get deployment nginx-deployment
```

**Output:**

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           59s
```

***

### Step 2: Attempt Update with Wrong Container Name

```bash
thor@jump-host ~$ kubectl set image deployment/nginx-deployment nginx=nginx:1.19
```

**Output:**

```
error: unable to find container named "nginx"
```

> **Observation:** The container name in the deployment is **not `nginx`**.

***

### Step 3: Identify the Correct Container Name

```bash
thor@jump-host ~$ kubectl get deployment nginx-deployment -o yaml | grep -i image
```

**Output:**

```
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx-app","type":"front-end"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"nginx-app"}},"strategy":{"type":"RollingUpdate"},"template":{"metadata":{"labels":{"app":"nginx-app"},"name":"nginx-replica"},"spec":{"containers":[{"image":"nginx:1.16","name":"nginx-container"}]}}}}
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
```

> **Observation:** The correct container name is `nginx-container`, and the current image is `nginx:1.16`.

***

### Step 4: Check Rollout Status (Before Update)

```bash
thor@jump-host ~$ kubectl rollout status deployment/nginx-deployment
```

**Output:**

```
deployment "nginx-deployment" successfully rolled out
```

***

### Step 5: Verify Pods (Before Update)

```bash
thor@jump-host ~$ kubectl get pods
```

**Output:**

```
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-fc677cbc9-4zmdh   1/1     Running   0          114s
nginx-deployment-fc677cbc9-vkkhj   1/1     Running   0          114s
nginx-deployment-fc677cbc9-z8t8s   1/1     Running   0          114s
```

***

### Step 6: Perform the Rolling Update

```bash
thor@jump-host ~$ kubectl set image deployment/nginx-deployment nginx-container=nginx:1.19
```

**Output:**

```
deployment.apps/nginx-deployment image updated
```

***

### Step 7: Monitor Rollout

```bash
thor@jump-host ~$ kubectl rollout status deployment/nginx-deployment
```

**Output:**

```
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment" successfully rolled out
```

***

### Step 8: Verify Deployment Image

```bash
thor@jump-host ~$ kubectl describe deployment nginx-deployment | grep Image
```

**Output:**

```
Image:         nginx:1.19
```

***

### Step 9: Verify Pods After Update

```bash
thor@jump-host ~$ kubectl get pods
```

**Output:**

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6655dc8cfb-8tmj9   1/1     Running   0          20s
nginx-deployment-6655dc8cfb-d92p8   1/1     Running   0          20s
nginx-deployment-6655dc8cfb-hsnrn   1/1     Running   0          26s
```

**Repeated check:**

```bash
thor@jump-host ~$ kubectl get pods
```

**Output:**

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6655dc8cfb-8tmj9   1/1     Running   0          49s
nginx-deployment-6655dc8cfb-d92p8   1/1     Running   0          49s
nginx-deployment-6655dc8cfb-hsnrn   1/1     Running   0          55s
```

***

### Result

* The **nginx-deployment** was successfully updated to **nginx:1.19**.
* All pods were **recreated gradually** via **rolling update**.
* Deployment remains fully **operational**, meeting task requirements.

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
