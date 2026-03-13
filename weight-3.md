# Weight: 3



A DevOps engineer attempted to deploy a Python application on the Kubernetes cluster. Unfortunately, due to misconfigurations, the application failed to launch. Your task is to investigate and rectify the issues.

Deployment and Service Details:

* Deployment Name: `python-deployment-devops-t4q4`
* Image Used: `poroko/flask-demo-app`

Ensure that the deployment and service configurations for this app are correctly deployed. Configure the `nodePort` to be `32345`, and set the `targetPort` to match the default port of the Python Flask application.

Your objective is to identify and correct any misconfigurations affecting the deployment and service of the Python application. Once rectified, verify the application's accessibility through the `App` button.

`Note:` The `kubectl` on `jump_host` has been configured to work with the kubernetes cluster.

Deployment 'python-deployment-devops-t4q4' exists

Service 'python-service-devops-t4q4' exists

Pods are in 'Running' state

Python app is up and accessible



***

### **Step 1️⃣: Create the Deployment YAML**

```bash
vi python-deployment-devops-t4q4.yaml
```

Paste the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-deployment-devops-t4q4
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
        - name: flask-container
          image: poroko/flask-demo-app
          ports:
            - containerPort: 5000
```

> ✅ Notes:
>
> * `matchLabels` in selector must match `labels` in the template.
> * `containerPort` = 5000 (default Flask port).

***

### **Step 2️⃣: Apply the Deployment**

```bash
kubectl apply -f python-deployment-devops-t4q4.yaml
```

Check that pods are running:

```bash
kubectl get pods -l app=python-app
```

You should see 2 pods in **Running** state.

***

### **Step 3️⃣: Create the Service YAML**

```bash
vi python-service-devops-t4q4.yaml
```

Paste the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-devops-t4q4
spec:
  type: NodePort
  selector:
    app: python-app
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 32345
```

> ✅ Notes:
>
> * `selector` matches pod label (`app: python-app`)
> * `nodePort` = 32345
> * `targetPort` = container port 5000

***

### **Step 4️⃣: Apply the Service**

```bash
kubectl apply -f python-service-devops-t4q4.yaml
```

Verify service:

```bash
kubectl get svc python-service-devops-t4q4
```

You should see:

```
NAME                        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
python-service-devops-t4q4  NodePort   <cluster-ip>    <none>        5000:32345/TCP   ...
```

***

### **Step 5️⃣: Verify Everything**

1. Check pods:

```bash
kubectl get pods -l app=python-app
```

2. Access the app:

* Use the **App button** in the lab environment
* Or open in browser: `http://<node-ip>:32345`

You should see the Flask application responding.

***

✅ **Result:**

* Deployment exists with 2 pods
* Pods are running
* Service exists with NodePort 32345
* Python Flask app is accessible

This resolves all previous errors (`selector immutable` and missing service file).

***

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
