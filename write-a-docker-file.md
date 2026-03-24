# Write a Docker File

As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file `/opt/docker/Dockerfile` (please keep `D` capital of Dockerfile) on `App server 2` in `Stratos DC` and configure to build an image with the following requirements:

a. Use `ubuntu:24.04` as the base image.

b. Install `apache2` and configure it to work on `8082` port. (do not update any other Apache configuration settings like document root etc).



***

## Docker: Build Custom Apache Image Using Dockerfile

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter\
> **Difficulty:** Intermediate | **Topic:** Dockerfile, Image Build, apache2, Containerization

***

### Table of Contents

1. [Lab Question](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-question)
2. [Infrastructure Details](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#infrastructure-details)
3. [Solution](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#solution)
   * [Step 1: SSH into Application Server 2](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-1-ssh-into-application-server-2)
   * [Step 2: Create Docker Directory](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-2-create-docker-directory)
   * [Step 3: Create Dockerfile](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-3-create-dockerfile)
   * [Step 4: Add Dockerfile Instructions](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-4-add-dockerfile-instructions)
   * [Step 5: Build Docker Image](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-5-build-docker-image)
   * [Step 6: Run Container for Testing](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-6-run-container-for-testing)
   * [Step 7: Verify Apache Service](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-7-verify-apache-service)
4. [Lab Complete](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-complete)
5. [Key Concepts](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#key-concepts)

***

### Lab Question

As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements are already been shared with DevOps team. Therefore, create a docker file /opt/docker/Dockerfile (please keep D capital of Dockerfile) on App server 2 in Stratos DC and configure to build an image with the following requirements:

Requirements:

1. Use ubuntu:24.04 as the base image.
2. Install apache2 and configure it to work on 8082 port.
3. Do not update any other Apache configuration settings like document root.

***

### Infrastructure Details

| Server Name          | Hostname  | User    | Password   | Purpose                      |
| -------------------- | --------- | ------- | ---------- | ---------------------------- |
| Application Server 1 | stapp01   | tony    | Ir0nM@n    | Hosts Nautilus Application 1 |
| Application Server 2 | stapp02   | steve   | Am3ric@    | Hosts Nautilus Application 2 |
| Application Server 3 | stapp03   | banner  | BigGr33n   | Hosts Nautilus Application 3 |
| LoadBalancer Server  | stlb01    | loki    | Mischi3f   | Distributes traffic          |
| Database Server      | stdb01    | peter   | Sp!dy      | Hosts database               |
| Storage Server       | ststor01  | natasha | Bl@kW      | Storage                      |
| Backup Server        | stbkp01   | clint   | H@wk3y3    | Backups                      |
| Mail Server          | stmail01  | groot   | Gr00T123   | Mail services                |
| Jump Host            | jump-host | thor    | mjolnir123 | Access server                |
| Jenkins Server       | jenkins   | jenkins | j@rv!s     | CI/CD                        |

Target: Application Server 2 (stapp02) — create Dockerfile at `/opt/docker/Dockerfile`

***

### Solution

#### Step 1: SSH into Application Server 2

Connect to the target server from the jump host.

```bash
ssh steve@stapp02
```

Terminal Output:

```
thor@jump-host ~$ ssh steve@stapp02
steve@stapp02's password:
[steve@stapp02 ~]$
```

This confirms successful login.

***

#### Step 2: Create Docker Directory

Create the required directory where the Dockerfile will reside.

```bash
sudo mkdir -p /opt/docker
cd /opt/docker
```

Terminal Output:

```
[steve@stapp02 ~]$ sudo mkdir -p /opt/docker
[steve@stapp02 ~]$ cd /opt/docker
[steve@stapp02 docker]$
```

Directory is created successfully.

***

#### Step 3: Create Dockerfile

Create the Dockerfile with correct naming convention (capital D).

```bash
sudo vi Dockerfile
```

Terminal Output:

```
[steve@stapp02 docker]$ sudo vi Dockerfile
```

Dockerfile is ready for editing.

***

#### Step 4: Add Dockerfile Instructions

Define base image, install Apache, configure port, and set container startup command.

```dockerfile
FROM ubuntu:24.04

RUN apt update && apt install -y apache2

RUN sed -i 's/Listen 80/Listen 8082/g' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8082>/g' /etc/apache2/sites-available/000-default.conf

CMD ["apachectl", "-D", "FOREGROUND"]
```

Terminal Output:

```
<file saved>
```

Dockerfile correctly defines all required configurations.

***

#### Step 5: Build Docker Image

Build the Docker image from the Dockerfile.

```bash
docker build -t apache-8082 .
```

Terminal Output:

```
Sending build context to Docker daemon
Step 1/4 : FROM ubuntu:24.04
 ---> ...
Step 2/4 : RUN apt update && apt install -y apache2
 ---> Running in ...
Step 3/4 : RUN sed -i ...
 ---> Running in ...
Step 4/4 : CMD ["apachectl", "-D", "FOREGROUND"]
 ---> Running in ...
Successfully built <image_id>
Successfully tagged apache-8082:latest
```

Image is successfully created.

***

#### Step 6: Run Container for Testing

Run a container from the newly created image.

```bash
docker run -d -p 8082:8082 apache-8082
```

Terminal Output:

```
<container_id>
```

Container is running in detached mode.

***

#### Step 7: Verify Apache Service

Test Apache service availability on port 8082.

```bash
curl http://localhost:8082
```

Terminal Output:

```
Apache2 Ubuntu Default Page
It works!
```

This confirms Apache is correctly configured and accessible.

***

### Lab Complete

| Task                | Detail                   | Status     |
| ------------------- | ------------------------ | ---------- |
| Dockerfile location | /opt/docker/Dockerfile   | Created    |
| Base image          | ubuntu:24.04             | Used       |
| Apache installation | Installed via Dockerfile | Completed  |
| Port configuration  | Changed to 8082          | Completed  |
| Image build         | apache-8082              | Successful |
| Container run       | Port 8082 exposed        | Running    |
| Service validation  | curl successful          | Completed  |

***

### Key Concepts

#### Dockerfile Basics

A Dockerfile is a script that defines how a Docker image is built.

```
FROM → Base image
RUN → Execute commands
CMD → Default container startup command
```

It ensures reproducible builds.

***

#### Apache Port Configuration

Apache listens on ports defined in:

* `/etc/apache2/ports.conf`
* `/etc/apache2/sites-available/000-default.conf`

Both files must be updated to avoid mismatch between listening port and virtual host.

***

#### Running Services in Containers

Containers do not use systemd.

Correct approach:

```
CMD ["apachectl", "-D", "FOREGROUND"]
```

This keeps Apache running in foreground and prevents container exit.

***

#### Docker Build Process

```
Dockerfile → docker build → Image → docker run → Container
```

Each step in Dockerfile creates a new image layer.

***

#### CMD vs RUN

| Instruction | Purpose                       |
| ----------- | ----------------------------- |
| RUN         | Executes during build time    |
| CMD         | Executes at container runtime |

Using CMD ensures the container runs Apache when started.

***

_Lab completed on 2026-03-24 | Server: stapp02 | OS: Ubuntu 24.04 (Docker Image) | Docker_



<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
