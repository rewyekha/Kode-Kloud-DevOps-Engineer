# Deploy Grafana on Kubernetes Cluster

The Nautilus DevOps teams is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on Kubernetes cluster. Below you can find more details.

1.) Create a deployment named `grafana-deployment-nautilus` using any grafana image for Grafana app. Set other parameters as per your choice.

2.) Create `NodePort` type service with nodePort `32000` to expose the app.

`You do not need to make any configuration changes inside the Grafana app once deployed; just make sure you can access the Grafana login page.`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



## Kubernetes: Deploy Grafana with NodePort Service

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Kubernetes, Grafana, Deployments, NodePort, Monitoring

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: Verify Cluster Node Status](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-verify-cluster-node-status)
   * [Step 2: Create the Grafana Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-create-the-grafana-deployment)
   * [Step 3: Verify the Deployment](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-verify-the-deployment)
   * [Step 4: Verify the Pod is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-the-pod-is-running)
   * [Step 5: Expose the Deployment as a NodePort Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-expose-the-deployment-as-a-nodeport-service)
   * [Step 6: Patch the Service to Set NodePort 32000](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-patch-the-service-to-set-nodeport-32000)
   * [Step 7: Verify the Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-verify-the-service)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus DevOps team is planning to set up a Grafana tool to collect and analyze analytics from some applications. They are planning to deploy it on a Kubernetes cluster.

**Requirements:**

1. Create a deployment named `grafana-deployment-nautilus` using any Grafana image. Set other parameters as per your choice.
2. Create a `NodePort` type service with `nodePort 32000` to expose the app.

You do not need to make any configuration changes inside the Grafana app once deployed — just make sure you can access the Grafana login page.

> **Note:** The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.

***

### Infrastructure Details

| Server Name          | Hostname    | User     | Password     | Purpose                            |
| -------------------- | ----------- | -------- | ------------ | ---------------------------------- |
| Application Server 1 | `stapp01`   | `tony`   | `Ir0nM@n`    | Hosts Nautilus Application 1       |
| Application Server 2 | `stapp02`   | `steve`  | `Am3ric@`    | Hosts Nautilus Application 2       |
| Application Server 3 | `stapp03`   | `banner` | `BigGr33n`   | Hosts Nautilus Application 3       |
| Jump Host            | `jump-host` | `thor`   | `mjolnir123` | Provides secure access to Stork DC |

> **Target:** All `kubectl` commands are executed from `jump-host`, which is pre-configured to communicate with the Kubernetes cluster.

***

### Solution

#### Step 1: Verify Cluster Node Status

Before creating any resources, confirm the cluster is healthy and the node is in `Ready` state:

```bash
kubectl get nodes
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get nodes
NAME        STATUS   ROLES           AGE   VERSION
jump-host   Ready    control-plane   29m   v1.34.1+k3s1
```

The cluster is a single-node k3s setup running `v1.34.1+k3s1` with the `jump-host` acting as the control plane. Status is `Ready` — the cluster is healthy and ready for workloads.

***

#### Step 2: Create the Grafana Deployment

Create the deployment using the official `grafana/grafana` image. The `kubectl create deployment` command creates a deployment with one replica by default:

```bash
kubectl create deployment grafana-deployment-nautilus \
  --image=grafana/grafana
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl create deployment grafana-deployment-nautilus \
--image=grafana/grafana
deployment.apps/grafana-deployment-nautilus created
```

The deployment `grafana-deployment-nautilus` was created using the `grafana/grafana` image. Kubernetes will pull the image and schedule the pod on an available node.

***

#### Step 3: Verify the Deployment

Check the deployment status immediately after creation:

```bash
kubectl get deployments
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get deployments
NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
grafana-deployment-nautilus   0/1     1            0           6s
```

At 6 seconds the deployment shows `READY: 0/1` — the pod is still being scheduled and the container image is being pulled. `UP-TO-DATE: 1` confirms the desired replica count is being worked toward.

***

#### Step 4: Verify the Pod is Running

Check the pod status to confirm the Grafana container started successfully:

```bash
kubectl get pods
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get pods
NAME                                          READY   STATUS    RESTARTS   AGE
grafana-deployment-nautilus-8db66d5b5-lnjl8   1/1     Running   0          13s
```

Pod `grafana-deployment-nautilus-8db66d5b5-lnjl8` is `1/1 Running` with `0` restarts at 13 seconds. The Grafana container started successfully and the application is ready to receive traffic.

***

#### Step 5: Expose the Deployment as a NodePort Service

Create a NodePort service to expose the Grafana app externally. Grafana listens on port `3000` by default:

```bash
kubectl expose deployment grafana-deployment-nautilus \
  --type=NodePort \
  --name=grafana-service-nautilus \
  --port=3000 \
  --target-port=3000
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl expose deployment grafana-deployment-nautilus \
--type=NodePort \
--name=grafana-service-nautilus \
--port=3000 \
--target-port=3000
service/grafana-service-nautilus exposed
```

The service `grafana-service-nautilus` of type `NodePort` was created. At this point Kubernetes assigned a random nodePort from the default range `30000-32767`. The next step sets it to the required `32000`.

***

#### Step 6: Patch the Service to Set NodePort 32000

The lab requires nodePort `32000` specifically. Use `kubectl patch` to update the service port configuration:

```bash
kubectl patch service grafana-service-nautilus \
  -p '{"spec":{"ports":[{"port":3000,"targetPort":3000,"nodePort":32000}]}}'
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl patch service grafana-service-nautilus \
-p '{"spec":{"ports":[{"port":3000,"targetPort":3000,"nodePort":32000}]}}'
service/grafana-service-nautilus patched
```

The service was patched successfully. The nodePort is now set to `32000` as required.

***

#### Step 7: Verify the Service

Confirm the service is configured correctly with the required port mapping:

```bash
kubectl get svc grafana-service-nautilus
```

**Terminal Output:**

```
thor@jump-host ~$ kubectl get svc grafana-service-nautilus
NAME                       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
grafana-service-nautilus   NodePort   10.43.168.161   <none>        3000:32000/TCP   19s
```

The service `grafana-service-nautilus` is confirmed as:

* Type: `NodePort`
* Cluster IP: `10.43.168.161`
* Port mapping: `3000:32000/TCP` — external traffic on nodePort `32000` is forwarded to container port `3000`

The Grafana login page is now accessible at `http://<node-ip>:32000`.

***

### Lab Complete

| Requirement        | Detail                                 | Status    |
| ------------------ | -------------------------------------- | --------- |
| Deployment name    | `grafana-deployment-nautilus`          | Confirmed |
| Image              | `grafana/grafana`                      | Confirmed |
| Pod status         | `1/1 Running`, `0 restarts`            | Confirmed |
| Service name       | `grafana-service-nautilus`             | Confirmed |
| Service type       | `NodePort`                             | Confirmed |
| Service port       | `3000`                                 | Confirmed |
| Target port        | `3000`                                 | Confirmed |
| NodePort           | `32000`                                | Confirmed |
| Grafana login page | Accessible at `http://<node-ip>:32000` | Confirmed |

***

### Key Concepts

#### Two-Step Service Configuration — expose then patch

The lab required a specific nodePort value of `32000`. The `kubectl expose` command does not directly accept a `--node-port` flag, so the workflow required two steps:

**Step 1:** `kubectl expose` — creates the service with a random nodePort assigned by Kubernetes.

**Step 2:** `kubectl patch` — updates the specific port field to the required value.

An alternative single-step approach is to write a service manifest YAML directly:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-nautilus
spec:
  type: NodePort
  selector:
    app: grafana-deployment-nautilus
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000
```

```bash
kubectl apply -f grafana-service.yaml
```

Both approaches produce the same result. The imperative approach (`expose` + `patch`) is faster for quick deployments while the declarative YAML approach is preferred for version-controlled production configurations.

#### NodePort Traffic Flow

```
Browser / External Client
         |
         | http://<node-ip>:32000
         v
Kubernetes Node (jump-host)
         |
         | NodePort 32000
         v
Service: grafana-service-nautilus
         |
         | Cluster IP 10.43.168.161:3000
         v
Pod: grafana-deployment-nautilus-8db66d5b5-lnjl8
         |
         | Container port 3000
         v
Grafana Application (login page)
```

#### kubectl patch — JSON Merge Patch

`kubectl patch` accepts a JSON or YAML snippet that is merged into the existing resource definition. The `-p` flag takes the patch as a string:

```bash
kubectl patch service grafana-service-nautilus \
  -p '{"spec":{"ports":[{"port":3000,"targetPort":3000,"nodePort":32000}]}}'
```

The patch replaces the entire `ports` array with the specified values. All three fields — `port`, `targetPort`, and `nodePort` — must be included when patching the ports array to avoid dropping existing values.

#### Grafana Default Credentials

The Grafana login page is accessible at the nodePort URL. Default credentials for a fresh Grafana deployment are:

| Field    | Default Value |
| -------- | ------------- |
| Username | `admin`       |
| Password | `admin`       |

Grafana prompts for a password change on first login. No further configuration was required for this lab — just confirming the login page loads confirms the deployment is healthy and the service routing is working correctly.

#### k3s vs Standard Kubernetes

The cluster runs `k3s v1.34.1` — a lightweight Kubernetes distribution designed for edge and development environments. It is fully compatible with standard `kubectl` commands and Kubernetes manifests. The single-node setup where `jump-host` acts as both the control plane and the worker node is typical for k3s lab environments.

***

_Lab completed on 2026-04-02 | Cluster: k3s v1.34.1 | Namespace: default | Pod: grafana-deployment-nautilus-8db66d5b5-lnjl8_

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>
