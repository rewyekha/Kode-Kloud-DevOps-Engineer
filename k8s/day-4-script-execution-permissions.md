# Day 4: Script Execution Permissions

The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization. Here are the details:

Create a pod named `httpd-pod` with a container named `httpd-container`. Use the `httpd` image with the `latest` tag (specify as `httpd:latest`). Set the following resource limits:

Requests: Memory: `15Mi`, CPU: `100m`

Limits: Memory: `20Mi`, CPU: `100m`

`Note:` The `kubectl` utility on `jump_host` is configured to operate with the Kubernetes cluster.



Here’s the exact YAML you need to create the pod with the specified resource requests and limits.

Create a file (e.g., `httpd-pod.yaml`) on the **jump\_host** and paste the following content:

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

#### Steps to apply it

1. Open a terminal on the **jump\_host** (you're already logged in as **thor** or whichever user the lab provides).
2.  Create the YAML file (you can use `vi`, `vim`, or `nano`):

    ```bash
    vi httpd-pod.yaml
    ```

    Paste the content above (in vi: press `i`, right-click to paste or Shift+Insert, then Esc, `:wq` to save and quit). Or with nano (easier):

    ```bash
    nano httpd-pod.yaml
    ```

    Paste, then Ctrl+O → Enter → Ctrl+X.
3.  Apply the pod definition:

    ```bash
    kubectl apply -f httpd-pod.yaml
    ```
4.  Verify the pod is created and check the resource limits:

    ```bash
    kubectl get pod httpd-pod
    ```

    ```bash
    kubectl describe pod httpd-pod
    ```

    In the output of `kubectl describe`, look under the container section — you should see:

    ```
    Requests:
      cpu:        100m
      memory:     15Mi
    Limits:
      cpu:        100m
      memory:     20Mi
    ```

That’s all! The pod will be created with exactly the requested resource requests and limits.

```
thor@jumphost ~$ vi httpd-pod.yaml
thor@jumphost ~$ kubectl apply -f httpd-pod.yaml
error: error parsing httpd-pod.yaml: error converting YAML to JSON: yaml: line 11: found character that cannot start any token
thor@jumphost ~$ vi httpd-pod.yaml
thor@jumphost ~$ kubectl apply -f httpd-pod.yaml
pod/httpd-pod created
thor@jumphost ~$ kubectl get pod httpd-pod
NAME        READY   STATUS    RESTARTS   AGE
httpd-pod   1/1     Running   0          16s
thor@jumphost ~$ kubectl describe pod httpd-pod
Name:             httpd-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kodekloud-control-plane/172.17.0.2
Start Time:       Fri, 19 Dec 2025 17:54:12 +0000
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  httpd-container:
    Container ID:   containerd://507c737e7bef08514402d3d5d849598101b2dc1127fb2ac833f94067166a5568
    Image:          httpd:latest
    Image ID:       docker.io/library/httpd@sha256:b913eada2685f101f93267e0984109966bbcc3afea6c9b48ed389afbf89863aa
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 19 Dec 2025 17:54:17 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  20Mi
    Requests:
      cpu:        100m
      memory:     15Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4xct5 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-4xct5:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  27s   default-scheduler  Successfully assigned default/httpd-pod to kodekloud-control-plane
  Normal  Pulling    26s   kubelet            Pulling image "httpd:latest"
  Normal  Pulled     22s   kubelet            Successfully pulled image "httpd:latest" in 4.371154535s (4.371169134s including waiting)
  Normal  Created    22s   kubelet            Created container httpd-container
  Normal  Started    22s   kubelet            Started container httpd-container
thor@jumphost ~$ 
```
