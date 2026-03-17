# Day 43: Docker Ports Mapping

The Nautilus DevOps team is planning to host an application on a nginx-based container. There are number of tickets already been created for similar tasks. One of the tickets has been assigned to set up a nginx container on `Application Server 3` in `Stratos Datacenter`. Please perform the task as per details mentioned below:

a. Pull `nginx:alpine` docker image on `Application Server 3`.

b. Create a container named `games` using the image you pulled.

c. Map host port `6200` to container port `80`. Please keep the container in running state.

### Task

The Nautilus DevOps team is planning to host an application on an **nginx-based container**. A ticket has been assigned to set up the container on **Application Server 3** in the Stratos Datacenter.

#### Requirements

1. Pull the Docker image **nginx:alpine** on **Application Server 3**.
2. Create a container named **games** using this image.
3. Map **host port 6200** to **container port 80**.
4. Ensure the container remains **running**.

***

## Infrastructure Details

| Server Name | Hostname                           | User   | Purpose        |
| ----------- | ---------------------------------- | ------ | -------------- |
| stapp03     | stapp03.stratos.xfusioncorp.com    | banner | Nautilus App 3 |
| jump\_host  | jump\_host.stratos.xfusioncorp.com | thor   | Jump Server    |

***

## Solution Steps

### Step 1: Login to Jump Host

```bash
ssh thor@jump_host.stratos.xfusioncorp.com
```

Password:

```
mjolnir123
```

***

### Step 2: Connect to Application Server 3

```bash
ssh banner@stapp03
```

Terminal output:

```
The authenticity of host 'stapp03 (10.244.29.206)' can't be established.
ED25519 key fingerprint is SHA256:0yksiW9GGWPbVxFrNBudR5CkAAcMIOAgCHCuFk2T4LQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
```

***

### Step 3: Pull the nginx Alpine Image

```bash
docker pull nginx:alpine
```

Terminal output:

```
alpine: Pulling from library/nginx
589002ba0eae: Pull complete
bca5d04786e1: Pull complete
3e2c181db1b0: Pull complete
6b7b6c7061b7: Pull complete
399d0898a94e: Pull complete
955a8478f9ac: Pull complete
6d397a54a185: Pull complete
5e7756927bef: Pull complete
Digest: sha256:1d13701a5f9f3fb01aaa88cef2344d65b6b5bf6b7d9fa4cf0dca557a8d7702ba
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```

***

### Step 4: Run the Container with Port Mapping

```bash
docker run -d -p 6200:80 --name games nginx:alpine
```

Terminal output:

```
fb566e89ebf5f58c13655635968586031127a3b9346cd5a3161af8834bed034e
```

Explanation:

* `-d` → Run container in detached mode
* `-p 6200:80` → Map host port **6200** to container port **80**
* `--name games` → Assign container name
* `nginx:alpine` → Docker image used

***

### Step 5: Verify the Running Container

```bash
docker ps
```

Terminal output:

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                                   NAMES
fb566e89ebf5   nginx:alpine   "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:6200->80/tcp, :::6200->80/tcp   games
```

***

## Verification

Test if nginx is accessible from the server.

```bash
curl http://localhost:6200
```

Expected output (HTML snippet):

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

***

## Result

* Docker image **nginx:alpine** successfully pulled.
* Container **games** created and running.
* Port mapping configured **6200 → 80**.
* nginx service accessible via **host port 6200**.



<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

