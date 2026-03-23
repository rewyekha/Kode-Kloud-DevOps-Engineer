# Weight: 6



The Nautilus devops team found that one of the applications that is deployed on the cluster is having some performance issues, they want to make some changes so that it can handle some more traffic. As per new updates some new changes need to be made in this existing setup. So update the deployment as per details mentioned below:

The deployment name is `blue-app-t2q5`, change its replicas count from `1` to `3`.

`Note:` The `kubectl` utility on `jump_host` has been configured to work with the kubernetes cluster.

Deployment 'blue-app-t2q5' exists

Pods are running

Deployment 'blue-app-t2q5' replicas count is '3'



#### Run this command:

```bash
kubectl scale deployment blue-app-t2q5 --replicas=3
```

***

#### Verify the update:

```bash
kubectl get deployment blue-app-t2q5
```

You should see something like:

```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
blue-app-t2q5    3/3     3            3           ...
```

Or check pods:

```bash
kubectl get pods
```

You should now see **3 pods** created by `blue-app-t2q5`.

***

Once:\
✔ Deployment exists\
✔ Replica count = 3\
✔ Pods are Running

You’re done ✅
