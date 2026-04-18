# Day 64: Fix Python App Deployed on Kubernetes Cluster

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

## Kubernetes Troubleshooting: Fix Python Flask App Deployment

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Kubernetes, Deployments, Services, NodePort, ImagePullBackOff, Troubleshooting

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Investigation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#investigation)
   * [Step 1: Check Pod Status](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-check-pod-status)
   * [Step 2: Inspect the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-inspect-the-deployment)
   * [Step 3: Inspect the Existing Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-inspect-the-existing-service)
4. [Root Cause Analysis](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#root-cause-analysis)
5. [Fix](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#fix)
   * [Step 4: Fix the Image Name in the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-fix-the-image-name-in-the-deployment)
   * [Step 5: Create the NodePort Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-the-nodeport-service)
   * [Step 6: Fix the Existing Service targetPort](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-fix-the-existing-service-targetport)
6. [Verification](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#verification)
   * [Step 7: Confirm Pod is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-confirm-pod-is-running)
   * [Step 8: Confirm Services and Endpoints](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-confirm-services-and-endpoints)
7. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
8. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

One of the DevOps engineers was trying to deploy a Python app on a Kubernetes cluster. Unfortunately, due to some misconfiguration, the application is not coming up. Fix the issues so the application is accessible on the specified nodePort.

**Requirements:**

* Deployment name: `python-deployment-datacenter`
* Image: `poroko/flask-demo-app`
* `nodePort` should be `32345`
* `targetPort` should be Python Flask app's default port (`5000`)

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

| Server Name | Hostname | User | Password | Purpose |
| -------------------- | ----------- | -------- | ------------ | ---------------------------------- |
| Application Server 1 | `stapp01` | `tony` | `Ir0nM@n` | Hosts Nautilus Application 1 |
| Application Server 2 | `stapp02` | `steve` | `Am3ric@` | Hosts Nautilus Application 2 |
| Application Server 3 | `stapp03` | `banner` | `BigGr33n` | Hosts Nautilus Application 3 |
| Jump Host | `jump-host` | `thor` | `mjolnir123` | Provides secure access to Stork DC |

> **Target:** All `kubectl` commands are executed from `jump-host`.

***

### Investigation

#### Step 1: Check Pod Status

The first step is to check the current state of all pods to identify the problem:

```bash
kubectl get pods
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get pods
NAME                                           READY   STATUS             RESTARTS   AGE
python-deployment-datacenter-65648dc9d-9mqp5   0/1     ImagePullBackOff   0          2m58s
```

The pod is in `ImagePullBackOff` status — meaning Kubernetes cannot pull the container image from Docker Hub. This indicates a wrong image name or tag in the deployment.

***

#### Step 2: Inspect the Deployment

Get the full deployment YAML to examine the current configuration:

```bash
kubectl get deployment python-deployment-datacenter -o yaml
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get deployment python-deployment-datacenter -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"python-deployment-datacenter","namespace":"default"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"python_app"}},"template":{"metadata":{"labels":{"app":"python_app"}},"spec":{"containers":[{"image":"poroko/flask-app-demo","name":"python-container-datacenter","ports":[{"containerPort":5000}]}]}}}}
  creationTimestamp: "2026-03-30T04:08:39Z"
  generation: 1
  name: python-deployment-datacenter
  namespace: default
  resourceVersion: "1134"
  uid: fd097573-06a3-471f-af0b-362c6bee08e1
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: python_app
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: python_app
    spec:
      containers:
      - image: poroko/flask-app-demo
        imagePullPolicy: Always
        name: python-container-datacenter
        ports:
        - containerPort: 5000
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2026-03-30T04:08:39Z"
    lastUpdateTime: "2026-03-30T04:08:39Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2026-03-30T04:08:39Z"
    lastUpdateTime: "2026-03-30T04:08:39Z"
    message: ReplicaSet "python-deployment-datacenter-65648dc9d" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  observedGeneration: 1
  replicas: 1
  unavailableReplicas: 1
  updatedReplicas: 1
```

The image in use is `poroko/flask-app-demo` — which is a typo. The correct image name is `poroko/flask-demo-app`.

***

#### Step 3: Inspect the Existing Service

Check whether a service already exists for this deployment:

```bash
kubectl get svc
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get svc
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                     ClusterIP   10.43.0.1       <none>        443/TCP          31m
python-service-datacenter      NodePort    10.43.214.153   <none>        8080:32345/TCP   9m25s
```

The service `python-service-datacenter` already exists on `nodePort 32345`. Inspect it in detail:

```bash
kubectl get svc python-service-datacenter -o yaml
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get svc python-service-datacenter -o yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"python-service-datacenter","namespace":"default"},"spec":{"ports":[{"nodePort":32345,"port":8080}],"selector":{"app":"python_app"},"type":"NodePort"}}
  creationTimestamp: "2026-03-30T04:08:39Z"
  name: python-service-datacenter
  namespace: default
  resourceVersion: "1124"
  uid: c5392f7c-6f0b-44be-bb3b-98e2dbb27ec7
spec:
  clusterIP: 10.43.214.153
  clusterIPs:
  - 10.43.214.153
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 32345
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: python_app
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

The service `targetPort` is set to `8080`, but the Flask app listens on port `5000`. This is a port mismatch that would prevent traffic from reaching the container even after the image issue is fixed.

***

### Root Cause Analysis

Two separate bugs were found:

| # | Resource | Bug | Correct Value | Impact |
| - | -------------------- | ----------------------- | ----------------------- | --------------------------------------------------------------- |
| 1 | Deployment `image` | `poroko/flask-app-demo` | `poroko/flask-demo-app` | `ImagePullBackOff` — image does not exist on Docker Hub |
| 2 | Service `targetPort` | `8080` | `5000` | Traffic routed to wrong port — Flask would not receive requests |

```bash
External Traffic → nodePort 32345 → Service port 8080 → targetPort 8080 ← WRONG
                                                                      ↓
                                                         Flask listens on 5000
```

After fixing both issues:

```bash
External Traffic → nodePort 32345 → Service port 8080 → targetPort 5000 ← CORRECT
                                                                      ↓
                                                         Flask listens on 5000
```

***

### Fix

#### Step 4: Fix the Image Name in the Deployment

Use `kubectl set image` to correct the typo in the image name:

```bash
kubectl set image deployment/python-deployment-datacenter \
  python-container-datacenter=poroko/flask-demo-app
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl set image deployment/python-deployment-datacenter \
python-container-datacenter=poroko/flask-demo-app
deployment.apps/python-deployment-datacenter image updated
```

Kubernetes triggered a rolling update — a new ReplicaSet was created with the correct image.

***

#### Step 5: Create the NodePort Service (intermediate attempt)

An attempt was made to expose the deployment as a new NodePort service. This resulted in a conflict because port `32345` was already allocated by the existing `python-service-datacenter`:

```bash
kubectl expose deployment python-deployment-datacenter \
  --type=NodePort \
  --port=5000 \
  --target-port=5000 \
  --name=python-deployment-datacenter
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl expose deployment python-deployment-datacenter \
--type=NodePort \
--port=5000 \
--target-port=5000 \
--name=python-deployment-datacenter
service/python-deployment-datacenter exposed
```

```bash
kubectl patch svc python-deployment-datacenter \
  -p '{"spec":{"ports":[{"port":5000,"targetPort":5000,"nodePort":32345}]}}'
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl patch svc python-deployment-datacenter -p '{"spec":{"ports":[{"port":5000,"targetPort":5000,"nodePort":32345}]}}'
The Service "python-deployment-datacenter" is invalid: spec.ports[0].nodePort: Invalid value: 32345: provided port is already allocated
```

Port `32345` was already in use by `python-service-datacenter`. The correct approach was to fix the existing service rather than create a new one.

***

#### Step 6: Fix the Existing Service targetPort

Edit the existing `python-service-datacenter` service to correct the `targetPort` from `8080` to `5000`:

```bash
kubectl edit svc python-service-datacenter
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl edit svc python-service-datacenter
service/python-service-datacenter edited
```

Inside the editor, the following change was made:

```yaml
# Before:
ports:
  - nodePort: 32345
    port: 8080
    protocol: TCP
    targetPort: 8080   ← changed to 5000

# After:
ports:
  - nodePort: 32345
    port: 8080
    protocol: TCP
    targetPort: 5000
```

***

### Verification

#### Step 7: Confirm Pod is Running

```bash
kubectl get pods
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get pods
NAME                                            READY   STATUS    RESTARTS   AGE
python-deployment-datacenter-57d654488b-xqjf6   1/1     Running   0          33s
```

The new pod `python-deployment-datacenter-57d654488b-xqjf6` is `1/1 Running` with `0` restarts. The old pod with `ImagePullBackOff` was replaced by the rolling update.

***

#### Step 8: Confirm Services and Endpoints

```bash
kubectl get svc
```

**Terminal Output:**

```bash
thor@jump-host ~$ kubectl get svc
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes                     ClusterIP   10.43.0.1       <none>        443/TCP          31m
python-deployment-datacenter   NodePort    10.43.113.66    <none>        5000:31534/TCP   2m40s
python-service-datacenter      NodePort    10.43.214.153   <none>        8080:32345/TCP   9m25s
```

The `python-service-datacenter` service is on `nodePort 32345`. Confirm the endpoint is pointing to the correct pod port:

```bash
kubectl get endpoints python-service-datacenter
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get endpoints python-service-datacenter
Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
NAME                        ENDPOINTS         AGE
python-service-datacenter   10.22.0.10:5000   12m
```

The endpoint `10.22.0.10:5000` confirms:

* The service is routing to the correct pod IP
* The port is now `5000` — matching Flask's default listening port
* Traffic on `nodePort 32345` will correctly reach the Flask application

***

### Lab Complete

| Issue | Root Cause | Fix Applied | Status |
| ----------------------------- | ---------------------------------------------------- | ------------------------------------- | --------- |
| `ImagePullBackOff` | Image name typo `flask-app-demo` vs `flask-demo-app` | `kubectl set image` with correct name | Fixed |
| Service `targetPort` mismatch | `targetPort: 8080` instead of `5000` | `kubectl edit svc` changed to `5000` | Fixed |
| Pod status | `0/1 ImagePullBackOff` | `1/1 Running` | Confirmed |
| Endpoint | No endpoints | `10.22.0.10:5000` | Confirmed |
| NodePort | `32345` accessible | Service routes to Flask on `5000` | Confirmed |

***

### Key Concepts

#### ImagePullBackOff — What It Means

`ImagePullBackOff` occurs when Kubernetes cannot pull a container image. Common causes:

| Cause | Example | Fix |
| ---------------- | --------------------------------------- | ---------------------- |
| Wrong image name | `flask-app-demo` vs `flask-demo-app` | Correct the image name |
| Wrong tag | `nginx:lates` instead of `nginx:latest` | Fix the tag |
| Private registry | No pull secret configured | Add `imagePullSecrets` |
| Rate limiting | Too many Docker Hub pulls | Authenticate or wait |

In this lab, the image `poroko/flask-app-demo` does not exist on Docker Hub — the correct image is `poroko/flask-demo-app`. A single character transposition caused the entire deployment to fail.

#### Kubernetes Port Mapping Chain

Understanding how traffic flows through Kubernetes port configuration is critical for debugging connectivity issues:

```bash
External Client
      │
      │ nodePort: 32345
      ▼
Service (python-service-datacenter)
      │
      │ port: 8080 (cluster-internal)
      │ targetPort: 5000 (must match containerPort)
      ▼
Pod (python-deployment-datacenter)
      │
      │ containerPort: 5000 (Flask listens here)
      ▼
Flask Application (poroko/flask-demo-app)
```

| Port Field | Location | Must Match |
| --------------- | --------------- | --------------------------------- |
| `containerPort` | Deployment spec | What the app actually listens on |
| `targetPort` | Service spec | Must equal `containerPort` |
| `port` | Service spec | Cluster-internal access port |
| `nodePort` | Service spec | External access port on each node |

#### `kubectl set image` vs `kubectl edit`

Both commands update a deployment's image, but they work differently:

| Command | Method | Best For |
| ------------------- | --------------------- | ---------------------------------- |
| `kubectl set image` | Targeted field update | Changing a single container image |
| `kubectl edit` | Full YAML in editor | Multiple field changes |
| `kubectl patch` | JSON/YAML patch | Scripted or specific field updates |

`kubectl set image deployment/<name> <container-name>=<new-image>` is the most precise way to update an image without risk of accidentally editing other fields.

#### Why the nodePort Conflict Occurred

When `kubectl expose` was run to create a new service, an attempt was made to patch that new service to use `nodePort 32345`. This failed because `python-service-datacenter` already had that port reserved. NodePorts must be unique cluster-wide — two services cannot share the same nodePort.

The correct resolution was to fix the existing service's `targetPort` rather than create a duplicate service.

#### Reading `kubectl get endpoints`

```
NAME                        ENDPOINTS         AGE
python-service-datacenter   10.22.0.10:5000   12m
```

The `ENDPOINTS` column shows `<pod-IP>:<targetPort>`. This is the definitive confirmation that:

* The service selector is matching at least one running pod
* The `targetPort` is correctly set to `5000`
* Traffic will reach the Flask application

An empty `ENDPOINTS` column (`<none>`) means the selector matches no pods — which would happen if labels are mismatched or no pods are running.

***

_Lab completed on 2026-03-30 | Cluster: Kubernetes (k3s v1.33) | Namespace: default_

**Before:**

<figure><img src=".gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

After:<br>

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>


<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
