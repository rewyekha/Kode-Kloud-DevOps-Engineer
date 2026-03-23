# Create Countdown Job in Kubernetes



The Nautilus DevOps team is crafting jobs in the Kubernetes cluster. While they're developing actual scripts/commands, they're currently setting up templates and testing jobs with dummy commands. Please create a job template as per details given below:

1. Create a job named `countdown-nautilus`.
2. The spec template should be named `countdown-nautilus` (under metadata), and the container should be named `container-countdown-nautilus`
3. Utilize image `debian` with `latest` tag (ensure to specify as `debian:latest`), and set the restart policy to `Never`.
4. Execute the command `sleep 5`

`Note:` The `kubectl` utility on `jump_host` is set up to operate with the Kubernetes cluster.

#### **Step 1: Log in to the jump\_host**

Make sure you are logged in to the jump host where `kubectl` is already configured.

```bash
ssh <jump_host>
```

***

#### **Step 2: Create a YAML file for the Job**

Create a new file named `countdown-nautilus.yaml`.

```bash
vi countdown-nautilus.yaml
```

***

#### **Step 3: Add the Job definition**

Paste the following content into the file:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdown-nautilus
spec:
  template:
    metadata:
      name: countdown-nautilus
    spec:
      restartPolicy: Never
      containers:
        - name: container-countdown-nautilus
          image: debian:latest
          command:
            - sleep
            - "5"
```

Save and exit the file.

***

#### **Step 4: Apply the Job to the Kubernetes cluster**

Run the following command to create the job:

```bash
kubectl apply -f countdown-nautilus.yaml
```

***

#### **Step 5: Verify the Job status**

Check that the job has been created:

```bash
kubectl get jobs
```

(Optional) View detailed information:

```bash
kubectl describe job countdown-nautilus
```

***

#### **Step 6: (Optional) Check the Pod created by the Job**

Find the pod created by the job:

```bash
kubectl get pods
```

Check pod logs (once it completes):

```bash
kubectl logs <pod-name>
```

***

✅ The `countdown-nautilus` job will run the command `sleep 5`, complete successfully, and will **not restart** due to the `Never` restart policy.

```bash
thor@jumphost ~$ vi countdown-nautilus.yaml
thor@jumphost ~$ kubectl apply -f countdown-nautilus.yaml
job.batch/countdown-nautilus created
thor@jumphost ~$ kubectl get jobs
NAME                 COMPLETIONS   DURATION   AGE
countdown-nautilus   0/1           8s         8s
thor@jumphost ~$ kubectl get pods
NAME                       READY   STATUS      RESTARTS   AGE
countdown-nautilus-dv77p   0/1     Completed   0          15s
thor@jumphost ~$ kubectl logs countdn-nautilus-dv77p
error: error from server (NotFound): pods "countdn-nautilus-dv77p" not found in namespace "default"
thor@jumphost ~$ kubectl logs countdown-nautilus-dv77p
thor@jumphost ~$ 
```
