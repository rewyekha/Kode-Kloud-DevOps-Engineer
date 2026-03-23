# Revert Deployment to Previous Version in Kubernetes

Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer has reported a bug related to this recent release. Consequently, the team aims to revert to the previous version.

There exists a deployment named `nginx-deployment`; initiate a rollback to the previous revision.

`Note:` The `kubectl` utility on `jump_host` is configured to interact with the Kubernetes cluster.

<pre><code><strong>thor@jumphost ~$ kubectl rollout undo deployment/nginx-deployment
</strong>deployment.apps/nginx-deployment rolled back
<strong>thor@jumphost ~$ kubectl rollout status deployment/nginx-deployment
</strong>deployment "nginx-deployment" successfully rolled out
<strong>thor@jumphost ~$ kubectl rollout history deployment/nginx-deployment
</strong>deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
2         kubectl set image deployment nginx-deployment nginx-container=nginx:stable --kubeconfig=/root/.kube/config --record=true
3         &#x3C;none>

thor@jumphost ~$ 
</code></pre>

### 🔁 Rollback Command

```bash
kubectl rollout undo deployment/nginx-deployment
```

***

### ✅ Verify the Rollback

Check rollout status:

```bash
kubectl rollout status deployment/nginx-deployment
```

(Optional) View rollout history:

```bash
kubectl rollout history deployment/nginx-deployment
```

***

#### ✔ Result

* The deployment is reverted to the **last stable revision**
* No configuration files need to be edited manually

If you want to roll back to a **specific revision number**, tell me and I’ll give you the exact command.
