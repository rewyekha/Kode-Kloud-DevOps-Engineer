# Resolve VolumeMounts Issue in Kubernetes

### Problem Statement

An Nginx and PHP-FPM setup deployed as a single Kubernetes Pod stopped working.\
Although the Pod was in a **Running** state, the website was not accessible.

The task was to:

1. Investigate the issue.
2. Fix the VolumeMount configuration.
3. Copy an `index.php` file into the Nginx document root.
4. Verify the website is accessible.

***

### Root Cause

The Pod contains **two containers**:

* `nginx-container`
* `php-fpm-container`

Both containers share an `emptyDir` volume, but **the volume was mounted at different paths**:

| Container         | Volume Mount Path       |
| ----------------- | ----------------------- |
| nginx-container   | `/var/www/html`         |
| php-fpm-container | `/usr/share/nginx/html` |

The Nginx configuration (`nginx-config` ConfigMap) defines the document root as:

```nginx
root /var/www/html;
```

Because PHP-FPM was not using the same directory, it could not access the PHP files Nginx was trying to execute.

***

### Solution Overview

* Recreate the Pod with **both containers mounting the shared volume at the same path**
* Use `/var/www/html` for **both Nginx and PHP-FPM**
* Copy the PHP file into the shared directory
* Verify the setup

***

### Fixed Pod Manifest

The Pod was recreated using the following manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  namespace: default
  labels:
    app: php-app
spec:
  containers:
  - name: php-fpm-container
    image: php:7.2-fpm-alpine
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html   # FIX: same path as nginx

  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-files
      mountPath: /var/www/html
    - name: nginx-config-volume
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf

  volumes:
  - name: shared-files
    emptyDir: {}
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

***

### Apply the Fix

#### Delete the existing Pod

```bash
kubectl delete pod nginx-phpfpm
```

#### Create the Pod with the corrected configuration

```bash
kubectl apply -f pod.yaml
```

***

### Copy PHP File into the Pod

Copy `index.php` from the jump host to the Nginx container:

```bash
kubectl cp /home/thor/index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
```

Since the volume is shared, PHP-FPM can also access this file.

***

### Verification

#### Test from inside the Nginx container

```bash
kubectl exec -it nginx-phpfpm -c nginx-container -- curl localhost:8099
```

Expected result:

* PHP page output is displayed
* No 404 or FastCGI errors

***

### Final Result

* ✅ VolumeMount paths aligned
* ✅ PHP files shared correctly between containers
* ✅ Nginx and PHP-FPM working together
* ✅ Website accessible using the **Website** button

***

### Key Takeaways

* Pod fields like `volumeMounts` are **immutable**
* To change them, the Pod must be **deleted and recreated**
* In multi-container Pods, **shared volumes must use the same mount path**
* Always match the application document root with the volume mount path

***

**Task completed successfully.** 🎉
