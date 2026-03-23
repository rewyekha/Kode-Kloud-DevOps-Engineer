# Day 2: Deploy Applications with Kubernetes Deployments

The Nautilus DevOps team is delving into Kubernetes for app management. One team member needs to create a deployment following these details:

Create a deployment named `nginx` to deploy the application `nginx` using the image `nginx:latest` (ensure to specify the tag)

`Note:` The `kubectl` utility on `jump_host` is set up to interact with the Kubernetes cluster.



***

### Steps to Create the Deployment

#### 1. Connect to the jump host

```bash
ssh thor@jump_host.stratos.xfusioncorp.com
```

Password:

```
mjolnir123
```

***

#### 2. Create the nginx Deployment

Run the following `kubectl` command:

```bash
kubectl create deployment nginx --image=nginx:latest
```

This command:

* Creates a deployment named **nginx**
* Uses the image **nginx:latest** (explicit tag specified)

***

#### 3. Verify the Deployment

Confirm that the deployment was created successfully:

```bash
kubectl get deployments
```

You should see output similar to:

```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           <time>
```

(Optional) Check the pods:

```bash
kubectl get pods
```

***

### ✅ Task Completed

* Deployment name: **nginx**
* Image used: **nginx:latest**
* Created using `kubectl` on **jump\_host**

```
thor@jumphost ~$ ssh thor@jump_host.stratos.xfusioncorp.com
The authenticity of host 'jump_host.stratos.xfusioncorp.com (172.19.0.2)' can't be established.
ED25519 key fingerprint is SHA256:CtkNUSeULzponMTDYeK2sO3tvk4fTOjVLMammU5ql0M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'jump_host.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
thor@jump_host.stratos.xfusioncorp.com's password: 
Last login: Wed Dec 17 15:15:36 2025

thor@jumphost ~$ kubectl create deployment nginx --image=nginx:latest
deployment.apps/nginx created
thor@jumphost ~$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           8s
thor@jumphost ~$ 
```
