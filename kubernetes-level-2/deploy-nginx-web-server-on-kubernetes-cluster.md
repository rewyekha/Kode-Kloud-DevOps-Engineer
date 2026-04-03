# Deploy Nginx Web Server on Kubernetes Cluster

The Nautilus team developers are working on a static website. They want it to be deployed on a Kubernetes cluster in a highly available and scalable manner. The DevOps team has decided to create a deployment with multiple replicas and expose it using a NodePort service.

***

### **Requirements**

1. **Deployment:**
   * Use the `nginx` image with the `latest` tag explicitly (`nginx:latest`).
   * Name the deployment `nginx-deployment`.
   * The container inside the deployment should be named `nginx-container`.
   * Set the replica count to **3** for high availability.
   * Expose container port `80`.
2. **Service:**
   * Create a Kubernetes Service of type **NodePort**.
   * Name the service `nginx-service`.
   * The NodePort should be set to **30011**.
   * The service should route traffic to the deployment pods.
3. **Testing:**
   * Verify the deployment and service are running.
   * Test external access using `curl` with the node’s IP and the specified NodePort.

***

## Deploying a Highly Available Nginx Static Website on Kubernetes

This guide demonstrates how to deploy a static website using Nginx on a Kubernetes cluster. The deployment is highly available, scalable, and exposed via a NodePort service.

***

### **Prerequisites**

* A Kubernetes cluster accessible from the jump-host.
* `kubectl` configured to interact with the cluster.
* Basic knowledge of Kubernetes deployments and services.

***

### **Step 1: Create the Deployment YAML**

We will create a Deployment named `nginx-deployment` with 3 replicas of the Nginx container using the `latest` tag.

Create a file named `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

**Explanation:**

* `replicas: 3` ensures high availability.
* `selector` and `labels` associate pods with the deployment.
* `containerPort: 80` exposes the container’s HTTP port.
* `image: nginx:latest` explicitly specifies the image and tag.

***

### **Step 2: Apply the Deployment**

Run the following command to create the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

**Expected Output:**

```bash
deployment.apps/nginx-deployment created
```

Verify the deployment status:

```bash
kubectl get deployments
```

**Expected Output:**

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           8s
```

Check the pods:

```bash
kubectl get pods -l app=nginx
```

**Expected Output:**

```bash
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6586c5b5fb-665v9   1/1     Running   0          18s
nginx-deployment-6586c5b5fb-sk99p   1/1     Running   0          18s
nginx-deployment-6586c5b5fb-wb7rj   1/1     Running   0          18s
```

***

### **Step 3: Create the NodePort Service YAML**

Create a service to expose the deployment externally via NodePort `30011`. Create a file named `nginx-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30011
```

**Explanation:**

* `type: NodePort` exposes the service on a static port across cluster nodes.
* `nodePort: 30011` is the external port for access.
* `selector: app: nginx` ensures the service routes traffic to the correct pods.

***

### **Step 4: Apply the Service**

Run the following command:

```bash
kubectl apply -f nginx-service.yaml
```

**Expected Output:**

```bash
service/nginx-service created
```

Verify the service:

```bash
kubectl get svc nginx-service
```

**Expected Output:**

```bash
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort   10.43.206.79   <none>        80:30011/TCP   5s
```

***

### **Step 5: Test the Deployment**

You can access the service via any node’s IP and the NodePort `30011`:

```bash
curl http://10.43.206.79:30011
```

**Expected Output (Nginx Default Page):**

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>
<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

This confirms that the deployment and service are fully functional.

***

### **Step 6: Scaling the Deployment (Optional)**

To increase or decrease replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods -l app=nginx
```

***

### **Conclusion**

* A highly available Nginx deployment has been created with 3 replicas.
* Exposed externally using a NodePort service on port 30011.
* Verified using `curl` to confirm the default Nginx page is served.

This setup ensures scalability and high availability for the static website.

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

