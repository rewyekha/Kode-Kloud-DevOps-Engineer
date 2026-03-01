# Day 36: Deploy Nginx Container on Application Server

The Nautilus DevOps team is conducting application deployment tests on selected application servers. They require a nginx container deployment on `Application Server 2`. Complete the task with the following instructions:

1. On `Application Server 2` create a container named `nginx_2` using the `nginx` image with the `alpine` tag. Ensure container is in a `running` state.



***

## Day 36: Deploy Nginx Container on Application Server 2

### 📌 Objective

Deploy a container named **`nginx_2`** using the **`nginx:alpine`** image on **Application Server 2 (`stapp02`)** and ensure it is running.

We will use Docker to deploy the **Nginx** container.

***

### 🖥 Infrastructure Details

| Server               | Hostname                          | User    |
| -------------------- | --------------------------------- | ------- |
| Application Server 2 | `stapp02.stratos.xfusioncorp.com` | `steve` |

***

### 🔹 Step 1: Connect to Application Server 2

Login from the jump host:

```bash
thor@jumphost ~$ ssh steve@stapp02
```

#### Terminal Output

```bash
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:+1uYhXI3+b4o+iol/Uzwf7o5AMDdnCf+g1V7DvPi2Tw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
```

After successful login:

```bash
[steve@stapp02 ~]$
```

***

### 🔹 Step 2: Pull Nginx Alpine Image

```bash
docker pull nginx:alpine
```

#### Terminal Output

```bash
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

### 🔹 Step 3: Run the Container

Create and start the container in detached mode:

```bash
docker run -d --name nginx_2 nginx:alpine
```

#### Terminal Output

```bash
e25f9b92288a819eb0884a82f0bb495ee1a2e1563f9346b18b920513031591bf
```

***

### 🔹 Step 4: Verify Container Status

```bash
docker ps
```

#### Terminal Output

```bash
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
e25f9b92288a   nginx:alpine   "/docker-entrypoint.…"   8 seconds ago   Up 7 seconds   80/tcp    nginx_2
```

***

### ✅ Verification

✔ Container name: `nginx_2`\
✔ Image used: `nginx:alpine`\
✔ Status: `Up` (Running)\
✔ Port exposed internally: `80/tcp`

***

### 🎯 Conclusion

The **nginx\_2** container has been successfully deployed on **Application Server 2 (`stapp02`)** and is running as expected.

***

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

