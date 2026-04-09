# Day 58: Deploy Grafana on Kubernetes Cluster

The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.

1.) Create a deployment named `grafana-deployment-xfusion` using any grafana image for Grafana app. Set other parameters as per your choice.

2.) Create `NodePort` type service with nodePort `32000` to expose the app.

`You do not need to make any configuration changes inside the Grafana app once deployed; just make sure you can access the Grafana login page.`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



## Grafana Deployment on Kubernetes

### Overview

This document describes the steps to deploy Grafana on a Kubernetes cluster and expose it using a NodePort service. The objective is to make the Grafana web interface accessible externally without modifying application-level configurations.

***

### Prerequisites

* Access to a configured Kubernetes cluster
* `kubectl` CLI installed and configured on the jump host
* Sufficient permissions to create deployments and services

***

### Step 1: Create Grafana Deployment

Create a deployment named `grafana-deployment-xfusion` using the official Grafana image.

```bash
thor@jump-host ~$ kubectl create deployment grafana-deployment-xfusion --image=grafana/grafana
deployment.apps/grafana-deployment-xfusion created
```

***

### Step 2: Verify Pod Status

Ensure that the Grafana pod is successfully created and running.

```bash
thor@jump-host ~$ kubectl get pods
NAME                                          READY   STATUS    RESTARTS   AGE
grafana-deployment-xfusion-5f586d9cf6-6b8ww   1/1     Running   0          12s
```

***

### Step 3: Expose Deployment via NodePort Service

Expose the deployment using a NodePort service to allow external access.

```bash
thor@jump-host ~$ kubectl expose deployment grafana-deployment-xfusion \
>   --type=NodePort \
>   --port=3000 \
>   --target-port=3000 \
>   --name=grafana-service-xfusion
service/grafana-service-xfusion exposed
```

***

### Step 4: Configure NodePort

Patch the service to use a fixed NodePort (`32000`).

```bash
thor@jump-host ~$ kubectl patch svc grafana-service-xfusion -p '{"spec":{"ports":[{"port":3000,"targetPort":3000,"nodePort":32000}]}}'
service/grafana-service-xfusion patched
```

***

### Step 5: Verify Service Configuration

Confirm that the service is exposed correctly and mapped to the desired NodePort.

```bash
thor@jump-host ~$ kubectl get svc
NAME                      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)          AGE
grafana-service-xfusion   NodePort    10.43.21.4   <none>        3000:32000/TCP   35s
kubernetes                ClusterIP   10.43.0.1    <none>        443/TCP          104m
```

***

### Step 6: Retrieve Node Information

Obtain the node IP address to access the Grafana interface.

```bash
thor@jump-host ~$ kubectl get nodes -o wide
NAME        STATUS   ROLES           AGE    VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
jump-host   Ready    control-plane   104m   v1.34.1+k3s1   10.244.195.199   <none>        Alpine Linux v3.16   6.8.0-94-generic   containerd://1.6.8
thor@jump-host ~$
```

***

### Step 7: Access Grafana

Access the Grafana web interface using the following URL:

```
http://<NODE-IP>:32000
```

Example:

```
http://10.244.195.199:32000
```

***

### Default Credentials

* Username: `admin`
* Password: `admin`

***

### Conclusion

Grafana has been successfully deployed on the Kubernetes cluster and exposed via a NodePort service. The application is accessible externally using the node IP and the configured port.



<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
