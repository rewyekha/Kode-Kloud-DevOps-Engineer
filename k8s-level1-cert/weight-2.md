# Weight: 2



One of our junior DevOps team members encountered an issue while deploying a stack on the Kubernetes cluster. The `webserver-t4q1` pod, with the `nginx-container` and a sidecar container named `sidecar-container`, is failing to start and remains in an error state.

Your task is to investigate and rectify the problem to ensure the successful running state of the `webserver-t4q1` pod. The `nginx-container` uses the `nginx:latest` image, while the sidecar-container utilizes the `ubuntu:latest` image. Ensure the `webserver-t4q1` pod is running as expected and the application is accessible.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.

Pod 'webserver-t4q1' exists

'nginx-container' exists

'sidecar-container' exists

'nginx-container' is using 'nginx:latest' image

Sidecar container is using 'ubuntu:latest' image

Pod 'webserver-t4q1' is up and running

Website is accessible



***

#### **Full Corrected Pod YAML**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver-t4q1
  labels:
    app: web-app
spec:
  containers:
    - name: nginx-container
      image: nginx:latest           # Fixed typo
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
    - name: sidecar-container
      image: ubuntu:latest
      command:
        - sh
        - -c
        - while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
  volumes:
    - name: shared-logs
      emptyDir: {}
```

***

#### **Step-by-Step Commands to Deploy**

1. **Delete the old failing pod** (if it exists):

```bash
kubectl delete pod webserver-t4q1
```

2. **Save the corrected YAML**:

```bash
vi webserver-t4q1.yaml
# Paste the full YAML above and save
```

3. **Apply the YAML**:

```bash
kubectl apply -f webserver-t4q1.yaml
```

4. **Verify the pod is running**:

```bash
kubectl get pods webserver-t4q1
```

You should see:

```
NAME             READY   STATUS    RESTARTS   AGE
webserver-t4q1   2/2     Running   0          <some seconds>
```

5. **Check logs**:

```bash
kubectl logs webserver-t4q1 -c nginx-container
kubectl logs webserver-t4q1 -c sidecar-container
```

* Sidecar logs may initially show “no file” until nginx writes logs.

6. **Test the application**:

* Use the **App button** in the lab environment or access via service/NodePort.

***

✅ **Outcome:**

* Both containers are running (`nginx-container` and `sidecar-container`)
* Shared volume `/var/log/nginx` allows the sidecar to access nginx logs
* Pod `webserver-t4q1` is healthy
* Website is accessible

***

```yaml
thor@jumphost ~$ kubectl get pods webserver-t4q1
Error from server (NotFound): pods "webserver-t4q1" not found
thor@jumphost ~$ vi webserver-t4q1.yaml
thor@jumphost ~$ kubectl apply -f webserver-t4q1.yaml
pod/webserver-t4q1 created
thor@jumphost ~$ kubectl get pods webserver-t4q1
NAME             READY   STATUS    RESTARTS   AGE
webserver-t4q1   2/2     Running   0          9s
thor@jumphost ~$ kubectl logs webserver-t4q1 -c nginx-container
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
thor@jumphost ~$ kubectl logs webserver-t4q1 -c sidecar-container
2026/02/23 08:15:59 [notice] 1#1: using the "epoll" event method
2026/02/23 08:15:59 [notice] 1#1: nginx/1.29.5
2026/02/23 08:15:59 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
2026/02/23 08:15:59 [notice] 1#1: OS: Linux 5.4.0-1106-gcp
2026/02/23 08:15:59 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/02/23 08:15:59 [notice] 1#1: start worker processes
2026/02/23 08:15:59 [notice] 1#1: start worker process 79
2026/02/23 08:15:59 [notice] 1#1: start worker process 80
2026/02/23 08:15:59 [notice] 1#1: start worker process 81
2026/02/23 08:15:59 [notice] 1#1: start worker process 82
2026/02/23 08:15:59 [notice] 1#1: start worker process 83
2026/02/23 08:15:59 [notice] 1#1: start worker process 84
2026/02/23 08:15:59 [notice] 1#1: start worker process 85
2026/02/23 08:15:59 [notice] 1#1: start worker process 86
2026/02/23 08:15:59 [notice] 1#1: start worker process 87
2026/02/23 08:15:59 [notice] 1#1: start worker process 88
2026/02/23 08:15:59 [notice] 1#1: start worker process 89
2026/02/23 08:15:59 [notice] 1#1: start worker process 90
2026/02/23 08:15:59 [notice] 1#1: start worker process 91
2026/02/23 08:15:59 [notice] 1#1: start worker process 92
2026/02/23 08:15:59 [notice] 1#1: start worker process 93
2026/02/23 08:15:59 [notice] 1#1: start worker process 94
thor@jumphost ~$ 
```

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

