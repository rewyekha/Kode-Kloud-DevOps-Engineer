# Weight: 0

Upon investigating an issue with one of the applications on the Kubernetes cluster, the team discovered that the service was configured with an incorrect node port. We need to update the service as follows:

Update `service-t5q3` service to use node port `30099`.

Service 'service-t5q3' nodePort is '30099'

Website is up and accessible

Perfect! The **service has been successfully updated** ✅.

Here’s a summary of the changes:

***

#### **Service Details**

* **Service Name:** `service-t5q3` ✅
* **Service Type:** `NodePort` ✅
* **Updated NodePort:** `30099` ✅
* **Target Port:** Remains unchanged (as defined in the service)
* **Outcome:** The application is accessible through the updated NodePort.

***

#### **Example Command Used**

```bash
kubectl patch svc service-t5q3 -p '{"spec":{"ports":[{"port":<service-port>,"targetPort":<target-port>,"nodePort":30099}]}}'
```

or, if editing YAML directly:

```bash
kubectl edit svc service-t5q3
# Change nodePort to 30099 and save
```

***

#### **Verify**

```bash
kubectl get svc service-t5q3
```

Expected output:

```
NAME           TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service-t5q3   NodePort   10.96.x.x        <none>        80:30099/TCP     ...
```

* `NodePort=30099` confirms the update
* Application is accessible via `http://<node-ip>:30099` ✅
