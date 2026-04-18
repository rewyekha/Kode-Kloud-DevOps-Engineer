# Day 39: Create a Docker Image From Container

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

a. Create an image `demo:xfusion` on `Application Server 2` from a container `ubuntu_latest` that is running on same server.

***

## Create a Docker Image from a Running Container

### Question

One of the Nautilus developers was testing new changes on a container and wants to keep a backup of those changes.

A request was raised for the DevOps team:

> Create a new image `demo:xfusion` on **Application Server 2** from the running container `ubuntu_latest`.

***

### Server Details

* Server: Application Server 2
* Hostname: `stapp02`
* Container Name: `ubuntu_latest`
* Target Image Name: `demo:xfusion`

***

### Solution Steps

#### 1 Connect to Application Server 2

```bash
thor@jumphost ~$ ssh steve@172.16.238.11
```

Output:

```
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:FXEmWaSmhw+FxJ4UK8jRySgwcIalaEld/LtN+w2vlsU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password:
```

After successful login:

```
[steve@stapp02 ~]$
```

***

#### 2 Verify Running Container

```bash
docker ps
```

Output:

```
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
c6af775a1447   ubuntu    "/bin/bash"   2 minutes ago   Up 2 minutes             ubuntu_latest
```

Filter specifically:

```bash
docker ps --filter "name=ubuntu_latest"
```

Output:

```
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
c6af775a1447   ubuntu    "/bin/bash"   2 minutes ago   Up 2 minutes             ubuntu_latest
```

***

#### 3 Create Image from Container

```bash
docker commit ubuntu_latest demo:xfusion
```

Output:

```
sha256:9b22064a7a8e3689dcc1a29c02c284df8a81eaf84a695da26976f7e28df0c053
```

***

#### 4 Verify Image Creation

```bash
docker images
```

Output:

```
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
demo         xfusion   9b22064a7a8e   12 seconds ago   138MB
ubuntu       latest    bbdabce66f1b   3 weeks ago      78.1MB
```

- Image `demo:xfusion` successfully created.

***

### What This Command Does

* `docker commit` creates a new image from a container’s current state.
* It includes:
  * File system changes
  * Installed packages
  * Configuration updates
* It does **not** include runtime data outside the container filesystem (e.g., mounted volumes).

***

## Common Mistakes

#### 1. Container Not Running

If you see no output from `docker ps`:

```
Error: No such container: ubuntu_latest
```

- Fix:

```bash
docker ps -a
docker start ubuntu_latest
```

***

#### 2. Wrong Container Name

Using incorrect name:

```bash
docker commit ubuntu demo:xfusion
```

This will fail if container name is different.

- Always confirm using:

```bash
docker ps
```

***

#### 3. Permission Denied

```
Got permission denied while trying to connect to the Docker daemon socket
```

- Fix:

```bash
sudo docker commit ubuntu_latest demo:xfusion
```

Or ensure the user is part of the docker group.

***

#### 4. Image Name Formatting Mistake

Wrong format:

```bash
docker commit ubuntu_latest demo xfusion
```

Correct format:

```bash
docker commit <container_name> <image_name>:<tag>
```

Example:

```bash
docker commit ubuntu_latest demo:xfusion
```

***

## Troubleshooting Guide

#### Verify Container Exists

```bash
docker ps -a
```

***

#### Check Docker Service Status

```bash
sudo systemctl status docker
```

If stopped:

```bash
sudo systemctl start docker
```

***

#### Confirm Image Creation

```bash
docker images | grep demo
```

Expected output:

```
demo   xfusion
```

***

## Final Verification Checklist

| Check | Status |
| --------------------------------- | ------ |
| Logged into Application Server 2 | |
| Container `ubuntu_latest` running | |
| Image `demo:xfusion` created | |
| Image visible in `docker images` | |

***

### Final Command Summary

```bash
ssh steve@172.16.238.11
docker ps
docker commit ubuntu_latest demo:xfusion
docker images
```

***

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
