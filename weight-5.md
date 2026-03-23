# Weight: 5



The Nautilus DevOps team is in the process of developing scripts to be executed on various schedules. Currently, they are provisioning cron jobs within the Kubernetes cluster with placeholder commands (to be substituted with actual scripts). Below are the specifications for creating a cronjob:

a. Create a cronjob named `devops-t3q1`.

b. Set Its schedule to something like `*/10 * * * *`, you set any schedule for now.

c. Container name should be `cron-devops-t3q1`.

d. Use `nginx` image with `latest tag` only and remember to mention the tag i.e `nginx:latest`.

e. Run a dummy command `echo Welcome to xfusioncorp!`.

f. Ensure restart policy is `OnFailure`.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.

Cronjob 'devops-t3q1' exists

Image used is 'nginx:latest'

Container name is 'cron-devops-t3q1'

Restart policy is set as 'OnFailure'



Create the CronJob using a manifest (recommended to control restartPolicy and container name exactly).

#### Create YAML

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: devops-t3q1
spec:
  schedule: "*/10 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cron-devops-t3q1
              image: nginx:latest
              command: ["/bin/sh", "-c"]
              args:
                - echo Welcome to xfusioncorp!
          restartPolicy: OnFailure
```

***

#### Apply it

```bash
kubectl apply -f devops-t3q1.yaml
```

***

#### Verify

Check cronjob exists:

```bash
kubectl get cronjobs
```

Check details:

```bash
kubectl describe cronjob devops-t3q1
```

Confirm:

✔ CronJob name = `devops-t3q1`\
✔ Image = `nginx:latest`\
✔ Container name = `cron-devops-t3q1`\
✔ RestartPolicy = `OnFailure`

Once verified, you're done ✅



```yaml
thor@jumphost ~$ kubectl describe cronjob devops-t3q1
Name:                          devops-t3q1
Namespace:                     default
Labels:                        <none>
Annotations:                   <none>
Schedule:                      */10 * * * *
Concurrency Policy:            Allow
Suspend:                       False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:     <unset>
Selector:                      <unset>
Parallelism:                   <unset>
Completions:                   <unset>
Pod Template:
  Labels:  <none>
  Containers:
   cron-devops-t3q1:
    Image:      nginx:latest
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      -c
    Args:
      echo Welcome to xfusioncorp!
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
  Node-Selectors:    <none>
  Tolerations:       <none>
Last Schedule Time:  <unset>
Active Jobs:         <none>
Events:              <none>
thor@jumphost ~$ 
```
