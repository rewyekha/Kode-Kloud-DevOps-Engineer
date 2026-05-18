# Deploy Node App on Kubernetes

The Nautilus development team has completed development of one of the node applications, which they are planning to deploy on a Kubernetes cluster. They recently had a meeting with the DevOps team to share their requirements. Based on that, the DevOps team has listed out the exact requirements to deploy the app. Find below more details:

1. Create a deployment using `kodekloud/centos-ssh-enabled:node` image, replica count must be `2`.
2. Create a service to expose this app, the service type must be `NodePort`, targetPort must be `8080` and nodePort should be `30012`.
3. Make sure all the pods are in `Running` state after the deployment.
4. You can check the application by clicking on `NodeApp` button on top bar.

`You can use any labels as per your choice.`

`Note:` The `kubectl` utility on the `jump-host` has been configured to work with the Kubernetes cluster.



***

## Deploying a Node Application on Kubernetes Using Deployment and NodePort Service

### Objective

The Nautilus development team completed the development of a Node.js application and requested deployment on a Kubernetes cluster. After reviewing the requirements shared with the DevOps team, the goal of this lab was to deploy the application using a Kubernetes **Deployment** and expose it externally using a **NodePort Service**.

This lab demonstrates creating a deployment with multiple replicas, exposing the application on a specific node port, and verifying that all pods are running successfully.

***

### Requirements

* Create a **Deployment** using the image:\
  `kodekloud/centos-ssh-enabled:node`
* Set the replica count to **2**
* Create a **Service** with the following specifications:
  * Type: `NodePort`
  * Target Port: `8080`
  * Node Port: `30012`
* Ensure all pods are in the **Running** state
* Confirm application accessibility using the **NodeApp** button from the UI

***

### Environment Details

* **Cluster Access**: Configured via `kubectl` on the `jump-host`
* **Namespace**: `default`
* **User**: `thor`
* **Kubernetes Utility**: Preconfigured and ready to use

***

### Implementation Steps

#### 1. Create the Deployment

A deployment named **`node-nautilus-deployment`** was created using the required image and replica count.

kubectl create deployment node-nautilus-deployment \</span>  --image=kodekloud/centos-ssh-enabled:node \</span>  --replicas=2

✅ **Result**: Deployment successfully created.

***

#### 2. Expose the Deployment (Initial Attempt)

An attempt was made to expose the deployment using the `kubectl expose` command with a specific nodePort. However, the `--node-port` flag is not supported directly with `kubectl expose`.

kubectl expose deployment node-nautilus-deployment \</span>  --name=node-nautilus-service \</span>  --type=NodePort \</span>  --port=80 \</span>  --target-port=8080 \</span>  --node-port=30012

❌ **Result**: Error received due to unsupported flag.

***

#### 3. Create the NodePort Service Using YAML

To meet the requirement of using a fixed nodePort (`30012`), the service was created using a YAML manifest.

cat <\<EOF | kubectl apply -f -apiVersion: v1kind: Servicemetadata:  name: node-nautilus-servicespec:  type: NodePort  selector:    app: node-nautilus-deployment  ports:    - port: 80      targetPort: 8080      nodePort: 30012EOF

✅ **Result**: Service successfully created and exposed.

***

### Verification

#### Verify Deployment Status

kubectl get deployment node-nautilus-deployment

✅ Output confirms:

* **2 replicas ready**
* Deployment available and up-to-date

***

#### Verify Pods Status

kubectl get pods\`\`

✅ Output confirms:

* Both pods are **Running**
* No restarts or failures

***

#### Pod Details Verification

kubectl describe pod node-nautilus-deployment-ff459b8dc-9mrvr

✅ Confirmed:

* Image pulled successfully
* Container running and ready
* Pod scheduled correctly

***

#### Verify Service Configuration

kubectl get svc node-nautilus-service

✅ Output confirms:

* Service type: **NodePort**
* Port mapping: `80:30012/TCP`

***

#### Verify Service Endpoints

kubectl get endpoints node-nautilus-service

✅ Output confirms:

* Endpoints mapped to both pod IPs on port `8080`

***

### Application Access

* The application is accessible through the Kubernetes node using **NodePort `30012`**
* The application can be tested directly via the **NodeApp button** in the lab interface

***

### Conclusion

The Node.js application was successfully deployed on the Kubernetes cluster using a Deployment with two replicas. A NodePort Service was created to expose the application on the required port, and all pods were verified to be in a healthy **Running** state.

This implementation fully satisfies the Nautilus development team’s requirements and demonstrates correct usage of Kubernetes Deployments and Services.

***

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>
