# Weight: 1



An application was previously deployed on the Kubernetes cluster, the deployment name is `deployment-t5q2`. This application is used by another applications within the same Kubernetes cluster. To enable access to this app, we require the creation of a `ClusterIP` service for the same.

Create a service named `deployment-svc-t5q2`. It must be a `ClusterIP` service which should use port `8090` and target port should be `80`.

Created service 'deployment-svc-t5q2'

Service type is 'ClusterIP'

Service 'deployment-svc-t5q2' port number is '8090'

Service 'deployment-svc-t5q2' target port is '80'



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

#### **Service Details**

* **Service Name:** `deployment-svc-t5q2` ✅
* **Type:** `ClusterIP` ✅
* **Port:** `8090` ✅
* **Target Port:** `80` ✅
* **Purpose:** Allows internal communication within the cluster to `deployment-t5q2` pods.

***

#### **Example YAML for reference**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: deployment-svc-t5q2
spec:
  type: ClusterIP
  selector:
    app: deployment-t5q2   # matches the labels of deployment-t5q2 pods
  ports:
    - port: 8090
      targetPort: 80
```

***

#### **Verify Service**

```bash
kubectl get svc deployment-svc-t5q2
kubectl describe svc deployment-svc-t5q2
```

You should see:

```
NAME                  TYPE        CLUSTER-IP       PORT(S)          AGE
deployment-svc-t5q2   ClusterIP   10.96.x.x        8090/TCP         ...
```

✅ **Result:** The service is correctly configured, and other applications in the cluster can access `deployment-t5q2` via `deployment-svc-t5q2:8090`.

```yaml
thor@jumphost ~$ vi deployment-svc-t5q2.yaml
thor@jumphost ~$ kubectl get svc deployment-svc-t5q2
Error from server (NotFound): services "deployment-svc-t5q2" not found
thor@jumphost ~$ kubectl apply -f deployment-svc-t5q2.yaml
service/deployment-svc-t5q2 created
thor@jumphost ~$ kubectl get svc deployment-svc-t5q2
NAME                  TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
deployment-svc-t5q2   ClusterIP   10.96.239.59   <none>        8090/TCP   11s
thor@jumphost ~$ kubectl describe svc deployment-svc-t5q2
Name:              deployment-svc-t5q2
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=deployment-t5q2
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.239.59
IPs:               10.96.239.59
Port:              <unset>  8090/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
