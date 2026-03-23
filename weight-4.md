# Weight: 4

The Nautilus DevOps team aims to establish a ReplicationController to deploy multiple pods for hosting applications requiring a highly available infrastructure. Here are the specific details to create the ReplicationController:

Create a `ReplicationController` using `nginx` image with `latest` tag, and name it as `nginx-replicationcontroller-t3q5`. All `pods` should be running state after deployment.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.

ReplicationController 'nginx-replicationcontroller-t3q5' exists

Image used is 'nginx'

Pods are 'Running'



create a **ReplicationController** using the `nginx:latest` image.

***

#### Create YAML

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-replicationcontroller-t3q5
spec:
  replicas: 3
  selector:
    app: nginx-t3q5
  template:
    metadata:
      labels:
        app: nginx-t3q5
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

***

#### Apply it

```bash
kubectl apply -f nginx-replicationcontroller-t3q5.yaml
```

***

#### Verify

Check the ReplicationController:

```bash
kubectl get rc
```

Check pods:

```bash
kubectl get pods
```

You should see **3 pods** in `Running` state.

To confirm image:

```bash
kubectl describe rc nginx-replicationcontroller-t3q5 | grep Image
```

***

Once:

✔ ReplicationController `nginx-replicationcontroller-t3q5` exists\
✔ Image used is `nginx:latest`\
✔ Pods are in `Running` state



```yaml
thor@jumphost ~$ vi nginx-replicationcontroller-t3q5.yaml
thor@jumphost ~$ kubectl apply -f nginx-replicationcontroller-t3q5.yamlreplicationcontroller/nginx-replicationcontroller-t3q5 created
thor@jumphost ~$ kubectl get rc
NAME                               DESIRED   CURRENT   READY   AGE
nginx-replicationcontroller-t3q5   3         3         0       7s
thor@jumphost ~$ kubectl get pods
NAME                                     READY   STATUS      RESTARTS   AGE
blue-app-t2q5-59cbfb78b7-c7ggc           1/1     Running     0          5m27s
blue-app-t2q5-59cbfb78b7-jq58l           1/1     Running     0          5m54s
blue-app-t2q5-59cbfb78b7-p8n2w           1/1     Running     0          5m27s
dummy-nginx-nginx-t1q6                   1/1     Running     0          11m
nginx-replicationcontroller-t3q5-9lt7h   1/1     Running     0          13s
nginx-replicationcontroller-t3q5-r7btq   1/1     Running     0          13s
nginx-replicationcontroller-t3q5-zs2vn   1/1     Running     0          13s
red-devops-t1q5                          0/1     Completed   0          21m
thor@jumphost ~$ kubectl describe rc nginx-replicationcontroller-t3q5 | grep Image
    Image:         nginx:latest
thor@jumphost ~$ 
```

