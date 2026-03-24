# Deploy Nginx Container on Application Server



***

## Deploying Nginx Container on Application Server 3

This guide walks you through deploying an **Nginx container using the Alpine image** on Application Server 3 (`stapp03`) for Nautilus DevOps application testing.

***

### 1. Connect to Application Server 3

First, SSH into the server from the jump host.

```bash
ssh banner@stapp03.stratos.xfusioncorp.com
```

**Expected Prompt:**

```
The authenticity of host 'stapp03.stratos.xfusioncorp.com (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXX.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

**Password:**

```
BigGr33n
```

**Common Mistakes:**

* **Wrong password**: Recheck capitalization and special characters. For `banner`, it is `BigGr33n`.
* **SSH Key Issues**: Ensure your jump host has access to the target server or password login is allowed.

***

### 2. Switch to Root (Optional but Recommended)

If your user has `sudo` privileges:

```bash
sudo -i
```

**Password:**

```
BigGr33n
```

Check Docker installation:

```bash
docker --version
```

**Expected Output:**

```
Docker version 26.1.3, build b72abbb
```

**Common Mistakes:**

* Docker not installed → Install Docker using the system’s package manager.
* Permission denied → Use `sudo` or ensure your user is in the `docker` group.

***

### 3. Pull the Nginx Alpine Image

```bash
docker pull nginx:alpine
```

**Expected Output:**

```
alpine: Pulling from library/nginx
...
Digest: sha256:1d13701a5f9f3fb01aaa88cef2344d65b6b5bf6b7d9fa4cf0dca557a8d7702ba
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
```

**Common Mistakes:**

* Typo in image name or tag (`nginx:alpine`) → Will cause `Error: image not found`.
* Network issues → Docker cannot pull the image if server cannot reach Docker Hub.

***

### 4. Run the Nginx Container

Create and start the container:

```bash
docker run -d --name nginx_3 nginx:alpine
```

**Explanation of Options:**

* `-d` → Run container in detached mode (background)
* `--name nginx_3` → Set a specific container name
* `nginx:alpine` → Image and tag

**Expected Output:**

```
4223f3f14738d13a806216d323f7e622248e18c6d71edc16aa14b18209d2e03e
```

***

### 5. Verify the Container is Running

```bash
docker ps
```

**Expected Output:**

```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
4223f3f14738   nginx:alpine   "/docker-entrypoint.…"   17 seconds ago  Up 16 seconds  80/tcp    nginx_3
```

**Common Mistakes:**

* Container not running → Check logs:

```bash
docker logs nginx_3
```

* Name conflicts → If a container named `nginx_3` already exists, remove it:

```bash
docker rm -f nginx_3
```

***

### ✅ Summary

* **Server:** stapp03
* **Container Name:** nginx\_3
* **Image:** nginx:alpine
* **Status:** Running

Your Nginx container is now successfully deployed and ready for testing.
