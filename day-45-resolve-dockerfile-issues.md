# Day 45: Resolve Dockerfile Issues
### Problem Statement
The Nautilus DevOps team needs to create a Docker image based on requirements provided by the development team. A team member attempted to build the image using a Dockerfile located on **App Server 1** but encountered errors during the build process.

#### Requirements
1. The **Dockerfile** is located on **App Server 1** in the directory:

```
/opt/docker
```

2. Investigate and **fix the Dockerfile so the image builds successfully**.
3. Do **not modify**:
   * Base image
   * Existing configuration logic
   * Data files (for example: `index.html`)
4. Once fixed, the Docker image must **build successfully without errors**.

***

## Infrastructure Details
| Server  | Hostname                        | User | Purpose               |
| ------- | ------------------------------- | ---- | --------------------- |
| stapp01 | stapp01.stratos.xfusioncorp.com | tony | Nautilus App Server 1 |

***

## Step 1: Connect to App Server
Login to the **jump host** and SSH into **App Server 1**.

```bash
ssh tony@stapp01
```

Terminal output:

```
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.13.38)' can't be established.
ED25519 key fingerprint is SHA256:GlMcQdbdiS5tzxGH4p2mWNly1NO4O5xkW9+zCKIRu9c.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
```

***

## Step 2: Navigate to Docker Directory
Move to the directory containing the Dockerfile.

```bash
cd /opt/docker
```

List files in the directory.

```bash
ls
```

Output:

```
Dockerfile  certs  html
```

***

## Step 3: Verify Existing Docker Images
Check if any Docker images already exist.

```bash
docker images
```

Output:

```
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

No images are present.

***

## Step 4: Inspect the Dockerfile
View the contents of the Dockerfile.

```bash
cat Dockerfile
```

Output:

```
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

RUN cp certs/server.crt /usr/local/apache2/conf/server.crt

RUN cp certs/server.key /usr/local/apache2/conf/server.key

RUN cp html/index.html /usr/local/apache2/htdocs/
```

#### Issue Identified
The Dockerfile uses:

```
RUN cp
```

However, Docker cannot copy files from the build context using `RUN cp`.
Instead, the **COPY instruction must be used**.

***

## Step 5: Edit the Dockerfile
Open the Dockerfile.

```bash
vi /opt/docker/Dockerfile
```

Replace the `RUN cp` commands with `COPY`.

#### Correct Dockerfile
```
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf

RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf/httpd.conf

RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf/httpd.conf

RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt

COPY certs/server.key /usr/local/apache2/conf/server.key

COPY html/index.html /usr/local/apache2/htdocs/index.html
```

Save and exit:

```
:wq
```

***

## Step 6: Build the Docker Image
Run the Docker build command.

```bash
docker build -t nautilus .
```

Output:

```
[+] Building 10.0s (13/13) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load metadata for docker.io/library/httpd:2.4.43
 => [internal] load build context
 => [2/8] RUN sed -i "s/Listen 80/Listen 8080/g"
 => [3/8] RUN sed -i '/LoadModule ssl_module/s/^#//g'
 => [4/8] RUN sed -i '/LoadModule socache_shmcb_module/s/^#//g'
 => [5/8] RUN sed -i '/Include conf/extra/httpd-ssl.conf/s/^#//g'
 => [6/8] COPY certs/server.crt
 => [7/8] COPY certs/server.key
 => [8/8] COPY html/index.html
 => exporting to image
 => naming to docker.io/library/nautilus
```

***

## Step 7: Verify the Image
Check that the image was successfully created.

```bash
docker images
```

Output:

```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
nautilus     latest    ee33baa9b053   14 seconds ago   166MB
```

***

## Final Result
The Dockerfile issue has been successfully resolved.

#### Key Fix
Replaced incorrect commands:

```
RUN cp
```

With the correct Docker instruction:

```
COPY
```

#### Outcome
* Dockerfile corrected
* Image built successfully
* Base image unchanged
* Application files preserved

Docker image **`nautilus:latest`** is now available.
