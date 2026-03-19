# Day 50: Set Resource Limits in Kubernetes Pods
### Overview
The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints.

Create a **pod named `httpd-pod`** with a container named **`httpd-container`** using the image **`httpd:latest`**.

Set the following resource configurations:

| Resource | Requests | Limits |
| -------- | -------- | ------ |
| Memory   | 15Mi     | 20Mi   |
| CPU      | 100m     | 100m   |

**Note:** The `kubectl` utility on the **jump-host** is already configured to work with the Kubernetes cluster.

***

## Solution
### Step 1: Connect to the Jump Host
```bash
ssh thor@jump-host
```

Enter password:

```
mjolnir123
```

***

## Step 2: Create Pod YAML File
Create the manifest file.

```bash
sudo vi httpd-pod.yaml
```

Add the following configuration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
      resources:
        requests:
          memory: "15Mi"
          cpu: "100m"
        limits:
          memory: "20Mi"
          cpu: "100m"
```

Save and exit.

***

## Step 3: Apply the Configuration
```bash
kubectl apply -f httpd-pod.yaml
```

Terminal Output:

```
pod/httpd-pod created
```

***

## Step 4: Verify Pod Status
```bash
kubectl get pods
```

Output:

```
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          16s
```

***

## Step 5: Verify Resource Requests and Limits
```bash
kubectl describe pod httpd-pod
```

Important section from output:

```
Containers:
  httpd-container:
    Image: httpd:latest
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:        100m
      memory:     15Mi
```

***

## Pod Details
| Property       | Value           |
| -------------- | --------------- |
| Pod Name       | httpd-pod       |
| Container Name | httpd-container |
| Image          | httpd:latest    |
| CPU Request    | 100m            |
| CPU Limit      | 100m            |
| Memory Request | 15Mi            |
| Memory Limit   | 20Mi            |
| Status         | Running         |

***

## Final Result
 Pod **httpd-pod** created successfully
 Container **httpd-container** running
 Resource **requests and limits configured correctly**
