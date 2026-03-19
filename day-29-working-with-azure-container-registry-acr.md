# Day 29: Working with Azure Container Registry (ACR)

The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a Azure Container Registry (ACR) to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the `azure-client` host and push this image to the ACR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

1\) Create a ACR repository named `datacenteracr8973` under `East US`.

2\) Pricing plan must be `Basic`.

3\) Dockerfile already exists under `/root/pyapp` directory on `azure-client` host.

4\) Build a Docker image using this Dockerfile and push the same to the newly created ACR repo. The image tag must be `latest` i.e `datacenteracr8973:latest`.

Use below given Azure Credentials: (You can run the `showcreds` command on `azure-client` host to retrieve these credentials)

| Portal URL | [https://portal.azure.com](https://portal.azure.com)                                                                                               |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Username   | [kk\_lab\_user\_main-9794630ed6cf4155@azurefreekmlprod.onmicrosoft.com](mailto:kk_lab_user_main-9794630ed6cf4155@azurefreekmlprod.onmicrosoft.com) |
| Password   | $L2                                                                                                                                                |
| Start Time | Tue Mar 03 06:38:59 UTC 2026                                                                                                                       |
| End Time   | Tue Mar 03 07:38:59 UTC 2026                                                                                                                       |

\
`Notes:`

* Create the resources only in `East US` region.
* To `display` or `hide` the terminal of the Azure client machine, you can use the expand toggle button as shown below:\
  ![toggle button](https://res.cloudinary.com/dezmljkdo/image/upload/v1678742174/AWS%20Lambda/expand_panel_hjgfkl.png)\
  <br>



***

## Working with Azure Container Registry (ACR) for Containerized Applications

### Overview

This guide covers the process of creating an Azure Container Registry (ACR), building a Docker image from your application, pushing the image to ACR, and verifying the image upload. It also explains the rationale behind each step, who consumes these container images, and the next actions a DevOps engineer should take after successful deployment.

***

### Prerequisites

* Azure Portal access and permissions to create resources.
* Azure CLI installed on your local or remote host.
* Docker installed on the build host.
* Application source code and Dockerfile available (e.g., under `/root/pyapp`).

***

## 1. Creating Azure Container Registry (ACR)

#### Why do we create an ACR?

ACR acts as a private Docker container registry to store and manage Docker container images securely within Azure. This enables teams to push, pull, and deploy images efficiently in the cloud environment.

#### Question:

**Q: What pricing tier did we choose and why?**\
**A:** We chose the Basic tier for cost efficiency in small or medium workloads with essential features.

<figure><img src=".gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

**Admin User:**

<figure><img src=".gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

***

## 2. Docker Image Build and Push Workflow

#### Step 1: Access application directory

```bash
~ ➜ cd /root/pyapp
~/pyapp ➜ ls
app.py  Dockerfile  requirements.txt
```

* The Dockerfile and application files are confirmed present.
* We need these files to build our Docker image.

***

#### Step 2: Login to Azure Container Registry (ACR)

```bash
~/pyapp ➜ docker login datacenteracr8973.azurecr.io -u datacenteracr8973 -p <ACR-Password>
WARNING! Using --password via the CLI is insecure. Use --password-stdin.

WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

* **Why this step?**\
  Docker needs authentication to push images to your private ACR.
* **Note:** Use the ACR login server URL (found in Azure Portal) as the registry URL.
* **Tip:** For better security, consider using `--password-stdin` instead of `-p` flag.

***

#### Step 3: Build the Docker Image

```bash
~/pyapp ➜ docker build -t datacenteracr8973.azurecr.io/datacenteracr8973:latest .
[+] Building 9.6s (9/9) FINISHED                                    docker:default
 => [internal] load build definition from Dockerfile                          0.1s
 => [1/4] FROM docker.io/library/python:3.8-slim@sha256:...                    4.9s
 => [4/4] RUN pip install -r requirements.txt                                 2.5s
 => exporting to image                                                        0.3s 
 => => naming to datacenteracr8973.azurecr.io/datacenteracr8973:latest        0.0s 
```

* The image is tagged with the ACR registry URL and `latest` tag.
* This image contains your application and its dependencies.

***

#### Step 4: Push Docker Image to ACR

```bash

~/pyapp ➜ docker push datacenteracr8973.azurecr.io/datacenteracr8973:latest
The push refers to repository [datacenteracr8973.azurecr.io/datacenteracr8973]
656c59c3a7c5: Pushed 
5f70bf18a086: Pushed 
...
latest: digest: sha256:e74b29ac67be5c2513752eed41335d40c78a16bcb68c51218f3b988dfac19458 size: 1783
```

* Push uploads the image layers to ACR.
* The image digest confirms a unique version of the image was saved.

<figure><img src=".gitbook/assets/image (35).png" alt=""><figcaption><p><strong>Before Push</strong></p></figcaption></figure>

<figure><img src=".gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>



<figure><img src=".gitbook/assets/image (37).png" alt=""><figcaption><p><strong>Built and Pushed Image</strong></p></figcaption></figure>

***

## 3. Verifying the Docker Image in ACR

#### List repositories

```bash
~/pyapp ➜ az acr repository list --name datacenteracr8973 --output table
Result
-----------------
datacenteracr8973
```

* Confirms repository presence in ACR.

#### List tags for the repository

```bash
~/pyapp ➜ az acr repository show-tags --name datacenteracr8973 --repository datacenteracr8973 --output table
Result
--------
latest
```

* Confirms the presence of the `latest` tag on the image.

#### Show image manifests (metadata)

```bash
~/pyapp ➜ az acr repository show-manifests --name datacenteracr8973 --repository datacenteracr8973 --output table
This command has been deprecated and will be removed in a future release. Use 'acr manifest list-metadata' instead.
Digest                                                                   Timestamp
-----------------------------------------------------------------------  ----------------------------
sha256:e74b29ac67be5c2513752eed41335d40c78a16bcb68c51218f3b988dfac19458  2026-03-03T06:53:55.0690142Z
```

* Provides detailed info on the pushed image versions.

***

## 4. How does this work in a live environment?

* The container image stored in ACR can be **pulled by Azure Kubernetes Service (AKS)**, Azure App Service, or any other orchestrator to run the application.
* Teams can automate deployment pipelines to **build → push → deploy** seamlessly.
* The registry acts as a **centralized hub** for container images, ensuring consistency and version control.

***

## 5. Who uses these containers?

* **Developers:** Pull the image to test or run locally.
* **DevOps Engineers:** Automate image builds and deploy containers on various environments.
* **Operations Teams:** Monitor containerized app performance and scale infrastructure.

***

## 6. Next Steps for a DevOps Engineer

* **Integrate ACR with a CI/CD pipeline:** Automate the image build and push process using Azure DevOps, GitHub Actions, or other tools.
* **Deploy containers to Kubernetes or Azure App Service:** Use the pushed images in production or staging environments.
* **Implement image scanning:** Scan for vulnerabilities before deployment for security best practices.
* **Setup Role-Based Access Control (RBAC):** Restrict who can push/pull images to the registry.
* **Enable geo-replication (if needed):** For global availability and redundancy.

***

## Summary

| Step                | Why?                                      | Terminal Output Example         |
| ------------------- | ----------------------------------------- | ------------------------------- |
| Login to ACR        | Authenticate Docker client to push images | `Login Succeeded`               |
| Build Docker Image  | Package app & dependencies                | `Successfully built <image_id>` |
| Push Docker Image   | Upload image to registry                  | `latest: digest: sha256:...`    |
| Verify Image in ACR | Confirm image presence and tags           | `Result: latest`                |

***

<figure><img src=".gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>
