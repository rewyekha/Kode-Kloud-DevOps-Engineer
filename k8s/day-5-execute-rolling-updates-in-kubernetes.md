# Day 5: Execute Rolling Updates in Kubernetes

An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image `nginx:1.17` with the latest updates.

Execute a rolling update for this application, integrating the `nginx:1.17` image. The deployment is named `nginx-deployment`.

Ensure all pods are operational post-update.

`Note:` The `kubectl` utility on `jump_host` is set up to operate with the Kubernetes cluster

<pre><code><strong>thor@jumphost ~$ kubectl get deployment nginx-deployment
</strong>NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           90s
<strong>thor@jumphost ~$ kubectl describe deployment nginx-deployment | grep -i image
</strong>    Image:         nginx:1.16
<strong>thor@jumphost ~$ kubectl set image deployment/nginx-deployment nginx=nginx:1.16
</strong>error: unable to find container named "nginx"
<strong>thor@jumphost ~$ kubectl get deployment nginx-deployment -o yaml | grep -i "name:"
</strong>  name: nginx-deployment
      name: nginx-replica
        name: nginx-container
      schedulerName: default-scheduler
thor@jumphost ~$ kubectl get deployment nginx-deployment -o=jsonpath='{.spec.template.spec.containers[*].name}'
nginx-containerthkubectl set image deployment/nginx-deployment nginx-container=nginx:1.17iner=nginx:1.17
^C
<strong>thor@jumphost ~$ kubectl set image deployment/nginx-deployment nginx-container=nginx:1.17
</strong>deployment.apps/nginx-deployment image updated
thor@jumphost ~$ kubectl rollout status deployment/nginx-deployment
deployment "nginx-deployment" successfully rolled out
<strong>thor@jumphost ~$ kubectl describe deployment nginx-deployment | grep -i image
</strong>    Image:         nginx:1.17
<strong>thor@jumphost ~$ kubectl get pods
</strong>NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5dd558cf95-78bv2   1/1     Running   0          45s
nginx-deployment-5dd558cf95-hmzqq   1/1     Running   0          37s
nginx-deployment-5dd558cf95-sdzsw   1/1     Running   0          39s
<strong>thor@jumphost ~$ 
</strong>
</code></pre>

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Step 1: Update the Image (Correct Container Name)

Run **exactly this**:

```bash
kubectl set image deployment/nginx-deployment nginx-container=nginx:1.17
```

This will trigger a **rolling update**.

***

### Step 2: Watch the Rolling Update

```bash
kubectl rollout status deployment/nginx-deployment
```

Wait until you see:

```
deployment "nginx-deployment" successfully rolled out
```

***

### Step 3: Verify the Image Version

```bash
kubectl describe deployment nginx-deployment | grep -i image
```

Expected output:

```
Image: nginx:1.17
```

***

### Step 4: Confirm All Pods Are Running

```bash
kubectl get pods
```

All pods should show:

```
STATUS: Running
```

***

### ✅ Final State (What the task requires)

* Deployment: **nginx-deployment**
* Container: **nginx-container**
* Image: **nginx:1.17**
* Update type: **Rolling update**
* Pods: **All Running (3/3)**

Once these checks pass, the **Nautilus application update is complete** 🚀
