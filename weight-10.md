# Weight: 10

In certain cases, applications deployed on the Kubernetes cluster require specific configurations or setup changes before launching the app container. The Nautilus DevOps team has devised a solution using init containers to fulfill these prerequisites during deployment. Below is an initial test scenario:

a. Create a Pod named `red-devops-t1q5`. It must have an init container named `red-init-devops-t1q5`, it should utilise image `debian` with the `latest` tag. The command '/bin/bash', '-c' should be used with arguments `echo "Welcome!"`

b. The main container name should be `red-main-devops-t1q5` and it should utilise image `debian` with the `latest` tag. The Command: '/bin/bash', '-c' should be used with arguments `sleep 1000`

This scenario demonstrates the use of `init` containers to fulfil pre-requisites before deploying the main application container in the Kubernetes Pod.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.\
\
checks:

Pod 'red-devops-t1q5' exits

Init container 'red-init-devops-t1q5' exists

Init container is utilising image 'debian'

Main container 'red-main-devops-t1q5' exists

Main container is using image 'debian'

Pod is 'Running'

Init container is configured as expected

Main container is configured as expected\
\
To meet the requirements, create a Pod manifest with an **init container** that runs first and prints `Welcome!`, followed by the main container that sleeps.

Below is the YAML definition:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-devops-t1q5
spec:
  initContainers:
    - name: red-init-devops-t1q5
      image: debian:latest
      command: ["/bin/bash", "-c"]
      args: ["echo \"Welcome!\""]
  containers:
    - name: red-main-devops-t1q5
      image: debian:latest
      command: ["/bin/bash", "-c"]
      args: ["sleep 1000"]
  restartPolicy: Never
```

#### Steps to Deploy

1.  Save the file on `jump_host`, for example:

    ```bash
    vi red-devops-t1q5.yaml
    ```
2.  Apply it:

    ```bash
    kubectl apply -f red-devops-t1q5.yaml
    ```
3.  Verify:

    ```bash
    kubectl get pods
    ```
4.  Check init container logs (optional):

    ```bash
    kubectl logs red-devops-t1q5 -c red-init-devops-t1q5
    ```

The init container will complete first (printing **Welcome!**), and only after that succeeds, the main container will start and sleep for 1000 seconds.\
\
✔ **Pod name:** `red-devops-t1q5`\
✔ **Init container name:** `red-init-devops-t1q5`\
✔ **Init container image:** `debian:latest` (logs confirm it executed)\
✔ **Init container command:** `/bin/bash -c "echo \"Welcome!\""` (output shows `Welcome!`)\
✔ **Main container name:** `red-main-devops-t1q5`\
✔ **Main container image:** `debian:latest`\
✔ **Main container command:** `/bin/bash -c "sleep 1000"`\
✔ **Pod status:** `Running`\
✔ **Init container completed successfully** (since main container is running)

Your `kubectl get pods` output shows:

```
READY   STATUS    RESTARTS
1/1     Running   0
```

That confirms:

* Init container executed and exited successfully.
* Main container is running.
* Pod is healthy.

🎯 **Conclusion: Done successfully.**

