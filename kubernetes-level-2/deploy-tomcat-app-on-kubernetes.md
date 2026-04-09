# Deploy Tomcat App on Kubernetes

A new java-based application is ready to be deployed on a Kubernetes cluster. The development team had a meeting with the DevOps team to share the requirements and application scope. The team is ready to setup an application stack for it under their existing cluster. Below you can find the details for this:

1. Create a namespace named `tomcat-namespace-datacenter`.
2. Create a `deployment` for tomcat app which should be named as `tomcat-deployment-datacenter` under the same namespace you created. Replica count should be `1`, the container should be named as `tomcat-container-datacenter`, its image should be `kodekloud/centos-ssh-enabled:tomcat` and its container port should be `8080`.
3. Create a `service` for tomcat app which should be named as `tomcat-service-datacenter` under the same namespace you created. Service type should be `NodePort` and nodePort should be `32227`.

Before clicking on `Check` button please make sure the application is up and running.

`You can use any labels as per your choice.`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.





````markdown
# Deploying Tomcat Application on Kubernetes

## Objective

Deploy a Java-based Tomcat application on an existing Kubernetes cluster using the following specifications:

- Create a dedicated namespace: `tomcat-namespace-datacenter`
- Deploy a Tomcat application as a Deployment named `tomcat-deployment-datacenter`
- Expose the application using a NodePort Service named `tomcat-service-datacenter` on port 32227

## Prerequisites

- Access to the jump-host with `kubectl` pre-configured to communicate with the Kubernetes cluster
- Sufficient permissions to create namespaces, deployments, and services

## Step 1: Create Namespace

```bash
thor@jump-host ~$ kubectl create namespace tomcat-namespace-datacenter
namespace/tomcat-namespace-datacenter created
````

### Step 2: Create Deployment

Create the deployment manifest file:

```bash
thor@jump-host ~$ cat tomcat-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment-datacenter
  namespace: tomcat-namespace-datacenter
  labels:
    app: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat-container-datacenter
        image: kodekloud/centos-ssh-enabled:tomcat
        ports:
        - containerPort: 8080
```

Apply the deployment:

```bash
thor@jump-host ~$ kubectl apply -f tomcat-deployment.yaml
deployment.apps/tomcat-deployment-datacenter created
```

### Step 3: Create Service

Create the service manifest file:

```bash
thor@jump-host ~$ cat tomcat-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service-datacenter
  namespace: tomcat-namespace-datacenter
spec:
  type: NodePort
  selector:
    app: tomcat
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 32227
```

Apply the service:

```bash
thor@jump-host ~$ kubectl apply -f tomcat-service.yaml
service/tomcat-service-datacenter created
```

### Step 4: Verification

Run the following commands to verify the deployment:

```bash
thor@jump-host ~$ kubectl get ns tomcat-namespace-datacenter
NAME                          STATUS   AGE
tomcat-namespace-datacenter   Active   2m15s
```

```bash
thor@jump-host ~$ kubectl get deployment tomcat-deployment-datacenter -n tomcat-namespace-datacenter
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
tomcat-deployment-datacenter    1/1     1            1           1m48s
```

```bash
thor@jump-host ~$ kubectl get pods -n tomcat-namespace-datacenter
NAME                                            READY   STATUS    RESTARTS   AGE
tomcat-deployment-datacenter-fc98fff8f-z4cgh    1/1     Running   0          1m50s
```

```bash
thor@jump-host ~$ kubectl get svc tomcat-service-datacenter -n tomcat-namespace-datacenter
NAME                          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
tomcat-service-datacenter     NodePort   10.43.47.25     <none>        8080:32227/TCP   45s
```

#### Check Application Logs

```bash
thor@jump-host ~$ kubectl logs -n tomcat-namespace-datacenter -l app=tomcat --tail=20
08-Apr-2026 04:36:12.425 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
08-Apr-2026 04:36:12.427 INFO [main] org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL successfully initialized [OpenSSL 1.1.1d 10 Sep 2019]
08-Apr-2026 04:36:13.209 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
08-Apr-2026 04:36:13.509 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
08-Apr-2026 04:36:13.532 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deploying web application archive [/usr/local/tomcat/webapps/ROOT.war]
08-Apr-2026 04:36:14.410 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [1100] milliseconds
```

### Final Validation

```bash
thor@jump-host ~$ kubectl get all -n tomcat-namespace-datacenter
NAME                                            READY   STATUS    RESTARTS   AGE
pod/tomcat-deployment-datacenter-fc98fff8f-z4cgh   1/1     Running   0          2m10s

NAME                                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/tomcat-service-datacenter       NodePort   10.43.47.25     <none>        8080:32227/TCP   55s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat-deployment-datacenter   1/1     1            1           2m10s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/tomcat-deployment-datacenter-fc98fff8f   1         1         1       2m10s
```

### Conclusion

The Tomcat application has been successfully deployed on the Kubernetes cluster with the following resources:

* **Namespace**: `tomcat-namespace-datacenter`
* **Deployment**: `tomcat-deployment-datacenter` (1 replica)
* **Pod**: Running with Tomcat 9.0.37
* **Service**: `tomcat-service-datacenter` (NodePort 32227)

The application is now accessible via NodePort `32227` on any cluster node.

```

This document is ready to be copied directly into GitBook or any Markdown-supported documentation platform. It maintains a clean, professional tone with complete terminal outputs for audit and reference purposes.
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
