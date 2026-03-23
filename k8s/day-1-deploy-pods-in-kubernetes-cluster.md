# Day 1: Deploy Pods in Kubernetes Cluster

The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:<br>

1. Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
2. Set the `app` label to `httpd_app`, and name the container as `httpd-container`.

`Note`: The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.



<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
```

You then apply this manifest using `kubectl` to create the pod. YAML manifests are the standard way to create Kubernetes pods via the _declarative approach_.([Kubernetes Tutorial](https://kubernetes-tutorial.schoolofdevops.com/create-and-deploy-kubernetes-pods/?utm_source=chatgpt.com))

***

### ✅ **Step 1 — Populate the YAML File Without an Editor**

Run this on the **jump\_host** shell where you already created `pod-httpd.yaml`:

```bash
cat > pod-httpd.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
EOF
```

**Explanation:**

* `cat > pod-httpd.yaml <<EOF` starts writing into the file.
* Everything up to the `EOF` line gets written into the file.
* This method works _without needing nano/vi/vim_. (based on **standard Kubernetes YAML usage**)([KodeKloud Notes](https://notes.kodekloud.com/docs/Docker-Certified-Associate-Exam-Course/Kubernetes/PODs-with-YAML?utm_source=chatgpt.com))

***

### ✅ **Step 2 — Verify the YAML Content**

To make sure the file contains the right text, run:

```bash
cat pod-httpd.yaml
```

You should see:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
```

***

### ✅ **Step 3 — Create the Pod in Kubernetes**

Now apply the manifest:

```bash
kubectl apply -f pod-httpd.yaml
```

If successful, you will see:

```
pod/pod-httpd created
```

***

### ✅ **Step 4 — Confirm the Pod Is Running**

Check the pod status:

```bash
kubectl get pods
```

You should see something like:

```
NAME         READY   STATUS    RESTARTS   AGE
pod-httpd    1/1     Running   0          10s
```

If it is still initializing, it might show `ContainerCreating` briefly before switching to `Running`.([KodeKloud Notes](https://notes.kodekloud.com/docs/Docker-Certified-Associate-Exam-Course/Kubernetes/PODs-with-YAML?utm_source=chatgpt.com))

***

#### 🧠 Quick Tips

* The label `app: httpd_app` is part of the metadata and can be used to select the pod later with `--selector`.([Kubernetes Tutorial](https://kubernetes-tutorial.schoolofdevops.com/create-and-deploy-kubernetes-pods/?utm_source=chatgpt.com))
* You explicitly used `httpd:latest` in the manifest — that satisfies the requirement to specify the **latest** tag.

