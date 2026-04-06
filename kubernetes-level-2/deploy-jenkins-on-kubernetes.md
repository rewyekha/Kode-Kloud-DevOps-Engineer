# Deploy Jenkins on Kubernetes

The Nautilus DevOps team is planning to set up a Jenkins CI server to create/manage some deployment pipelines for some of the projects. They want to set up the Jenkins server on Kubernetes cluster. Below you can find more details about the task:

1\) Create a namespace `jenkins`

2\) Create a Service for jenkins deployment. Service name should be `jenkins-service` under `jenkins` namespace, type should be `NodePort`, nodePort should be `30008`

3\) Create a Jenkins Deployment under `jenkins` namespace, It should be name as `jenkins-deployment` , labels `app` should be `jenkins` , container name should be `jenkins-container` , use `jenkins/jenkins` image , containerPort should be `8080` and replicas count should be `1`.

Make sure to wait for the pods to be in running state and make sure you are able to access the Jenkins login screen in the browser before hitting the `Check` button.

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

## Setting up Jenkins CI on Kubernetes

### Objective

The Nautilus DevOps team aims to deploy a Jenkins CI server on a Kubernetes cluster to manage deployment pipelines. This document outlines the step-by-step process, including namespace creation, service configuration, deployment setup, and validation.

***

### Step 1: Create Namespace

```bash
thor@jump-host ~$ kubectl create namespace jenkins
namespace/jenkins created
```

***

### Step 2: Create Jenkins Service

```bash
thor@jump-host ~$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
  namespace: jenkins
spec:
  type: NodePort
  selector:
    app: jenkins
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30008
EOF
service/jenkins-service created
```

***

### Step 3: Create Jenkins Deployment

```bash
thor@jump-host ~$ cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
  namespace: jenkins
  labels:
    app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins-container
          image: jenkins/jenkins
          ports:
            - containerPort: 8080
EOF
deployment.apps/jenkins-deployment created
```

***

### Step 4: Verify Rollout Status

```bash
thor@jump-host ~$ kubectl rollout status deployment/jenkins-deployment -n jenkins
Waiting for deployment "jenkins-deployment" rollout to finish: 0 of 1 updated replicas are available...
deployment "jenkins-deployment" successfully rolled out
```

***

### Step 5: Check Pod Status

```bash
thor@jump-host ~$ kubectl get pods -n jenkins
NAME READY STATUS RESTARTS AGE
jenkins-deployment-749b885fbb-7xx8k 1/1 Running 0 18s
```

***

### Step 6: Inspect Logs

```bash
thor@jump-host ~$ kubectl logs -f deployment/jenkins-deployment -n jenkins
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
...
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

ab22792f934548789fc40632e1a2d888

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```

***

### Step 7: Retrieve Initial Admin Password

```bash
thor@jump-host ~$ kubectl exec -n jenkins -it deployment/jenkins-deployment -- cat /var/jenkins_home/secrets/initialAdminPassword
ab22792f934548789fc40632e1a2d888
```

***

### Step 8: Restart Deployment (Optional)

```bash
thor@jump-host ~$ kubectl rollout restart deployment/jenkins-deployment -n jenkins
deployment.apps/jenkins-deployment restarted

thor@jump-host ~$ kubectl rollout status deployment/jenkins-deployment -n jenkins
deployment "jenkins-deployment" successfully rolled out
```

***

### Step 9: Cleanup (Optional)

```bash
thor@jump-host ~$ kubectl delete deployment jenkins-deployment -n jenkins --force --grace-period=0 2>/dev/null || true
deployment.apps "jenkins-deployment" force deleted from jenkins namespace

kubectl delete service jenkins-service -n jenkins --force --grace-period=0 2>/dev/null || true
service "jenkins-service" force deleted from jenkins namespace

kubectl delete namespace jenkins --force --grace-period=0 2>/dev/null || true
namespace "jenkins" force deleted
```

***

### Step 10: Recreate Namespace and Deployment (Validation)

```bash
thor@jump-host ~$ kubectl create namespace jenkins
namespace/jenkins created

thor@jump-host ~$ kubectl apply -f service.yaml
service/jenkins-service created

thor@jump-host ~$ kubectl apply -f deployment.yaml
deployment.apps/jenkins-deployment created

thor@jump-host ~$ kubectl rollout status deployment/jenkins-deployment -n jenkins --timeout=180s
deployment "jenkins-deployment" successfully rolled out

thor@jump-host ~$ kubectl get pods -n jenkins
NAME READY STATUS RESTARTS AGE
jenkins-deployment-749b885fbb-9vtm4 1/1 Running 0 15s
```

***

### Step 11: Access Jenkins

The Jenkins service is exposed via NodePort `30008`. Access Jenkins login screen in the browser using:

```
http://<Node-IP>:30008
```

***

### Conclusion

The Jenkins CI server was successfully deployed on Kubernetes. The namespace, service, and deployment were created, pods reached the running state, and Jenkins was accessible via NodePort. The initial admin password was retrieved to complete the setup.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
