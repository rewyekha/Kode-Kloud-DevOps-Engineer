# Day 53: Resolve VolumeMounts Issue in Kubernetes

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:

The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Identify and fix the problem.

Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root. After this, you should be able to access the website using `Website` button on the top bar.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.


***

## Debugging Nginx + PHP-FPM on Kubernetes

### Lab Question

> We encountered an issue with our Nginx and PHP-FPM setup on the Kubernetes cluster this morning, which halted its functionality. Investigate and rectify the issue:
>
> The pod name is `nginx-phpfpm` and the ConfigMap name is `nginx-config`. Identify and fix the problem.
>
> Once resolved, copy `/home/thor/index.php` file from the jump host to the nginx container within the nginx document root. After this, you should be able to access the website using the **Website** button on the top bar.
>
> **Note:** the `kubectl` utility on the jump-host has been configured to work with the Kubernetes cluster.

***

### Step 1: Check current pod and ConfigMap

```bash
thor@jump-host ~$ kubectl get pods
No resources found in default namespace
```

Check pod description (if it existed):

```bash
thor@jump-host ~$ kubectl describe pod nginx-phpfpm
# Output truncated; shows nginx + php-fpm containers running, shared volume mounted
```

Check nginx logs:

```bash
thor@jump-host ~$ kubectl logs nginx-phpfpm -c nginx-container
# Output shows normal docker-entrypoint logs; no PHP served yet
```

View ConfigMap:

```bash
thor@jump-host ~$ kubectl get configmap nginx-config -o yaml
apiVersion: v1
data:
  nginx.conf: |
    events {
    }
    http {
      server {
        listen 8099 default_server;
        listen [::]:8099 default_server;
        root /var/www/html;
        index index.html index.htm index.php;
        server_name _;
        location / { try_files $uri $uri/ =404; }
        location ~ \.php$ {
          include fastcgi_params;
          fastcgi_param REQUEST_METHOD $request_method;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_pass 127.0.0.1:9000;
        }
      }
    }
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
```

***

### Step 2: Fix ConfigMap

Edit ConfigMap:

```bash
thor@jump-host ~$ kubectl edit configmap nginx-config
```

Make sure:

```nginx
listen 8099 default_server;
listen [::]:8099 default_server;
root /var/www/html;
```

Save changes (`ESC` → `:wq` → Enter).

***

### Step 3: Recreate the pod

Create `pod.yaml`:

```bash
thor@jump-host ~$ vi pod.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  labels:
    app: php-app
spec:
  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
  containers:
  - name: php-fpm-container
    image: php:7.2-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
```

Apply the pod:

```bash
thor@jump-host ~$ kubectl apply -f pod.yaml
pod/nginx-phpfpm created
```

Check pod status:

```bash
thor@jump-host ~$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          6s
```

***

### Step 4: Copy `index.php` into nginx document root

```bash
thor@jump-host ~$ kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

Verify inside pod:

```bash
thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- ls /var/www/html
index.php
```

***

### Step 5: Test PHP-FPM connection

```bash
thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- curl http://127.0.0.1:8099/index.php
# Output: shows PHP page content
```

***

### Step 6: Access the website

Click the **Website button** on the top bar — the page should now load successfully without 502 errors.

***

### Key Notes

1. Both nginx and php-fpm containers **must share the same volume path** `/var/www/html`.
2. ConfigMap must point to **the same document root** as the volume.
3. nginx listening port should match the lab expectation (**8099** in this case).
4. Pod needs to be **recreated** after ConfigMap changes to take effect.
5. Copy the application file after pod is running.

***

### Terminal Output Summary

```bash
thor@jump-host ~$ kubectl delete pod nginx-phpfpm
pod "nginx-phpfpm" deleted from default namespace

thor@jump-host ~$ kubectl apply -f pod.yaml
pod/nginx-phpfpm created

thor@jump-host ~$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
nginx-phpfpm   2/2     Running   0          6s

thor@jump-host ~$ kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container

thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- ls /var/www/html
index.php

thor@jump-host ~$ kubectl exec -it nginx-phpfpm -c nginx-container -- curl http://127.0.0.1:8099/index.php
<?php phpinfo(); ?>
```

***


<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
