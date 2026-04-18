# Day 56: Deploy Nginx Web Server on Kubernetes Cluster

Some of the Nautilus team developers are developing a static website and they want to deploy it on Kubernetes cluster. They want it to be highly available and scalable. Therefore, based on the requirements, the DevOps team has decided to create a deployment for it with multiple replicas. Below you can find more details about it:

1. Create a deployment using `nginx` image with `latest` tag only and remember to mention the tag i.e `nginx:latest`. Name it as `nginx-deployment`. The container should be named as `nginx-container`, also make sure replica counts are `3`.
2. Create a `NodePort` type service named `nginx-service`. The nodePort should be `30011`.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

## Kubernetes Deployment and NodePort Service for NGINX

### Overview

This document describes the step-by-step procedure to:

* Create a Kubernetes Deployment using the `nginx:latest` image
* Configure the Deployment with 3 replicas
* Set the container name to `nginx-container`
* Expose the Deployment via a NodePort Service
* Configure the NodePort as `30011`

***

### Prerequisites

* Access to a Kubernetes cluster
* `kubectl` configured on the jump host
* Sufficient permissions to create and edit resources

***

### Step 1: Create Deployment

Run the following command to create the deployment:

```bash
kubectl create deployment nginx-deployment \
  --image=nginx:latest \
  --replicas=3
```

#### Output

```bash
deployment.apps/nginx-deployment created
```

***

### Step 2: Edit Deployment to Set Container Name

Edit the deployment:

```bash
kubectl edit deployment nginx-deployment
```

Locate the container section:

```yaml
containers:
- name: nginx
  image: nginx:latest
```

Modify it as follows:

```yaml
containers:
- name: nginx-container
  image: nginx:latest
```

Save and exit the editor.

#### Output

```bash
deployment.apps/nginx-deployment edited
```

***

### Step 3: Verify Deployment

Check deployment status:

```bash
kubectl get deployments
```

#### Output

```bash
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment    3/3     3            3           <time>
```

Check pods:

```bash
kubectl get pods
```

#### Output

```bash
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-864f55c6cf-f6xl9   1/1     Running   0          <time>
nginx-deployment-864f55c6cf-ml62d   1/1     Running   0          <time>
nginx-deployment-864f55c6cf-n8xkz   1/1     Running   0          <time>
```

Verify container name:

```bash
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].name}'
```

#### Output

```bash
nginx-container
```

***

### Step 4: Create NodePort Service

Expose the deployment:

```bash
kubectl expose deployment nginx-deployment \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80
```

#### Output

```bash
service/nginx-service exposed
```

***

### Step 5: Edit Service to Set NodePort

Edit the service:

```bash
kubectl edit svc nginx-service
```

Locate:

```yaml
ports:
- nodePort: 30463
  port: 80
  protocol: TCP
  targetPort: 80
```

Update it to:

```yaml
ports:
- nodePort: 30011
  port: 80
  protocol: TCP
  targetPort: 80
```

Save and exit.

#### Output

```bash
service/nginx-service edited
```

***

### Step 6: Verify Service

```bash
kubectl get svc nginx-service
```

#### Output

```bash
NAME            TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.43.103.212   <none>        80:30011/TCP   <time>
```

***

### Complete YAML Manifest (Alternative Approach)

The following YAML can be used to create both Deployment and Service in a single step.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-deployment
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30011
```

Apply the file:

```bash
kubectl apply -f nginx.yaml
```

***

### Troubleshooting

#### Error: "no original object found for Service"

Cause:

* Attempted to paste Service YAML inside `kubectl edit deployment`

Resolution:

* Edit only Deployment fields when using `kubectl edit deployment`
* Create Service separately

***

#### Error: "unknown flag: --node-port"

Cause:

* Some Kubernetes versions do not support `--node-port` with `kubectl expose`

Resolution:

* Use `kubectl edit svc nginx-service` to manually set `nodePort`

***

#### Error: "provided port is already allocated"

Cause:

* NodePort already in use

Resolution:

```bash
kubectl delete svc nginx-service
```

Recreate service and assign a different port or retry 30011 if available.

***

### Conclusion

The Deployment and Service have been successfully configured with:

* Deployment name: `nginx-deployment`
* Image: `nginx:latest`
* Replicas: 3
* Container name: `nginx-container`
* Service name: `nginx-service`
* Service type: NodePort
* NodePort: 30011

The application is now highly available and accessible via the NodePort.

***

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>
