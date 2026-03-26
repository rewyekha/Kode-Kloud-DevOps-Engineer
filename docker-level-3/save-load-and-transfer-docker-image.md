# Save, Load and Transfer Docker Image

One of the DevOps team members was working on to create a new custom docker image on `App Server 1` in `Stratos DC`. He is done with his changes and image is saved on same server with name `games:datacenter`. Recently a requirement has been raised by a team to use that image for testing, but the team wants to test the same on `App Server 3`. So we need to provide them that image on `App Server 3` in `Stratos DC`.

a. On `App Server 1` save the image `games:datacenter` in an archive.

b. Transfer the image archive to `App Server 3`.

c. Load that image archive on `App Server 3` with same name and tag which was used on `App Server 1`.

`Note:` Docker is already installed on both servers; however, if its service is down please make sure to start it.



## Docker: Save, Transfer, and Load a Docker Image Between Servers

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker, Image Management, docker save, docker load, SCP

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 1](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-1)
   * [Step 2: Verify the Source Image Exists](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-verify-the-source-image-exists)
   * [Step 3: Save the Image as a tar Archive](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-save-the-image-as-a-tar-archive)
   * [Step 4: Verify the Archive Was Created](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-the-archive-was-created)
   * [Step 5: Transfer the Archive to App Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-transfer-the-archive-to-app-server-3)
   * [Step 6: SSH into Application Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-ssh-into-application-server-3)
   * [Step 7: Verify Docker Service on App Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-verify-docker-service-on-app-server-3)
   * [Step 8: Confirm No Images Exist Yet on App Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-confirm-no-images-exist-yet-on-app-server-3)
   * [Step 9: Load the Image Archive](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-9-load-the-image-archive)
   * [Step 10: Verify the Image on App Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-10-verify-the-image-on-app-server-3)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

One of the DevOps team members was working on creating a new custom Docker image on **App Server 1** in Stratos DC. He is done with his changes and the image is saved on the same server with name `games:datacenter`. Recently a requirement has been raised by a team to use that image for testing, but the team wants to test the same on **App Server 3**. The image needs to be provided on App Server 3 in Stratos DC.

**Requirements:**

* **a.** On App Server 1, save the image `games:datacenter` in an archive.
* **b.** Transfer the image archive to App Server 3.
* **c.** Load that image archive on App Server 3 with the same name and tag which was used on App Server 1.

> **Note:** Docker is already installed on both servers. However, if its service is down please make sure to start it.

***

### Infrastructure Details

| Server Name          | Hostname    | User      | Password     | Purpose                               |
| -------------------- | ----------- | --------- | ------------ | ------------------------------------- |
| Application Server 1 | `stapp01`   | `tony`    | `Ir0nM@n`    | Hosts Nautilus Application 1          |
| Application Server 2 | `stapp02`   | `steve`   | `Am3ric@`    | Hosts Nautilus Application 2          |
| Application Server 3 | `stapp03`   | `banner`  | `BigGr33n`   | Hosts Nautilus Application 3          |
| LoadBalancer Server  | `stlb01`    | `loki`    | `Mischi3f`   | Distributes traffic for Nautilus HTTP |
| Database Server      | `stdb01`    | `peter`   | `Sp!dy`      | Hosts Nautilus Database               |
| Storage Server       | `ststor01`  | `natasha` | `Bl@kW`      | Stores data for Nautilus Servers      |
| Backup Server        | `stbkp01`   | `clint`   | `H@wk3y3`    | Manages backups for Nautilus Servers  |
| Mail Server          | `stmail01`  | `groot`   | `Gr00T123`   | Manages email services                |
| Jump Host            | `jump-host` | `thor`    | `mjolnir123` | Provides secure access to Stork DC    |
| Jenkins Server       | `jenkins`   | `jenkins` | `j@rv!s`     | Runs Jenkins for CI/CD pipeline       |

> **Source server:** `stapp01` — image `games:datacenter` exists here. **Destination server:** `stapp03` — image must be loaded here with identical name and tag.

***

### Solution

#### Step 1: SSH into Application Server 1

From the jump host, connect to `stapp01` as user `tony`:

```bash
ssh tony@stapp01
```

**Terminal Output:**

```
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.73.135)' can't be established.
ED25519 key fingerprint is SHA256:+ykkwFrSzoKLnYvYEEvTeto1FIttbGkMAP9ksvTYxR0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 2: Verify the Source Image Exists

Before saving, confirm that the `games:datacenter` image exists on App Server 1. Also check the `/tmp` directory which contains a helper script:

```bash
ls /tmp
```

**Terminal Output:**

```
[tony@stapp01 ~]$ ls /tmp
docker_container.sh
systemd-private-5f028c47d25d40ec86cdf7e8025eaa42-dbus-broker.service-ZaZU9o
systemd-private-5f028c47d25d40ec86cdf7e8025eaa42-rtkit-daemon.service-ChPrRs
systemd-private-5f028c47d25d40ec86cdf7e8025eaa42-systemd-logind.service-MeivxI
systemd-private-5f028c47d25d40ec86cdf7e8025eaa42-upower.service-MK2jf7
```

The `/tmp` directory contains `docker_container.sh` — a pre-existing script used to build the `games:datacenter` image. The image itself is already built and available in the local Docker image cache.

***

#### Step 3: Save the Image as a tar Archive

Use `docker save` to export the `games:datacenter` image into a portable `.tar` archive file stored at `/tmp/games.tar`:

```bash
docker save -o /tmp/games.tar games:datacenter
```

**Terminal Output:**

```
[tony@stapp01 ~]$ docker save -o /tmp/games.tar games:datacenter
[tony@stapp01 ~]$
```

No output means the command completed successfully. The `-o` flag specifies the output file path. The archive contains all image layers, metadata, and the repository tag information needed to restore the image with its original name and tag on another host.

***

#### Step 4: Verify the Archive Was Created

Confirm the archive file exists and check its size:

```bash
ls -l /tmp/games.tar
```

**Terminal Output:**

```
[tony@stapp01 ~]$ ls -l /tmp/games.tar
-rw------- 1 tony tony 141432832 Mar 26 03:53 /tmp/games.tar
```

The archive file `games.tar` is `141,432,832 bytes` (approximately 135MB). The file permissions `rw-------` mean only the owner `tony` can read and write it. This is the default behaviour of `docker save -o`.

***

#### Step 5: Transfer the Archive to App Server 3

Use `scp` (Secure Copy Protocol) to transfer the archive file from `stapp01` to `stapp03`. The file is transferred to the `/tmp` directory on the destination server:

```bash
scp /tmp/games.tar banner@stapp03:/tmp/
```

**Terminal Output:**

```
[tony@stapp01 ~]$ scp /tmp/games.tar banner@stapp03:/tmp/
The authenticity of host 'stapp03 (10.244.244.229)' can't be established.
ED25519 key fingerprint is SHA256:4rbQlMPqes312wpXSlf0HGW31sC+GhvzHruVhraWTaU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
games.tar                                100%  135MB 107.9MB/s   00:01
```

Enter password: `BigGr33n`

The 135MB archive transferred at 107.9MB/s and completed in 1 second. The progress bar confirms `100%` transfer completion.

***

#### Step 6: SSH into Application Server 3

Connect directly to `stapp03` from `stapp01` to load the image. Note: the first password attempt failed due to a typo — the second attempt succeeded:

```bash
ssh banner@stapp03
```

**Terminal Output:**

```
[tony@stapp01 ~]$ ssh banner@stapp03
banner@stapp03's password:
Permission denied, please try again.
banner@stapp03's password:
Last failed login: Thu Mar 26 03:57:27 UTC 2026 from 10.244.73.135 on ssh:notty
There was 1 failed login attempt since the last successful login.
[banner@stapp03 ~]$
```

Enter password: `BigGr33n`

> The first password attempt failed and was retried. This is noted in the terminal history and has no impact on the lab result.

***

#### Step 7: Verify Docker Service on App Server 3

The lab note states Docker service should be verified and started if it is down. Check the Docker service status:

```bash
systemctl status docker
```

**Terminal Output:**

```
[banner@stapp03 ~]$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-03-26 03:28:18 UTC; 29min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1380 (dockerd)
      Tasks: 21
     Memory: 29.9M (peak: 31.8M)
        CPU: 333ms
     CGroup: /system.slice/docker.service
             └─1380 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Mar 26 03:28:17 stapp03 systemd[1]: Starting Docker Application Container Engine...
Mar 26 03:28:17 stapp03 dockerd[1380]: time="2026-03-26T03:28:17.467028425Z" level=info msg="Starting up"
Mar 26 03:28:17 stapp03 dockerd[1380]: time="2026-03-26T03:28:17.778512724Z" level=info msg="Loading containers: start."
Mar 26 03:28:18 stapp03 dockerd[1380]: time="2026-03-26T03:28:18.008383158Z" level=info msg="Loading containers: done."
Mar 26 03:28:18 stapp03 dockerd[1380]: level=warning msg="Not using native diff for overlay2, this may cause degraded performance"
Mar 26 03:28:18 stapp03 dockerd[1380]: level=info msg="Docker daemon" commit=8e96db1
Mar 26 03:28:18 stapp03 dockerd[1380]: level=info msg="Daemon has completed initialization"
Mar 26 03:28:18 stapp03 dockerd[1380]: level=info msg="API listen on /run/docker.sock"
Mar 26 03:28:18 stapp03 systemd[1]: Started Docker Application Container Engine.
```

Docker service is `active (running)` since `03:28:18 UTC` with Main PID `1380`. No action was needed to start it — it was already running.

> Note: An initial attempt was made with `systemctl docker status` which returned `Unknown command verb docker`. The correct syntax is `systemctl status docker` — with the service name after `status`.

***

#### Step 8: Confirm No Images Exist Yet on App Server 3

Before loading, verify the destination server has no pre-existing images:

```bash
docker images
```

**Terminal Output:**

```
[banner@stapp03 ~]$ docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

The image list is empty — confirming `games:datacenter` does not yet exist on `stapp03`.

Also confirm no containers are running:

```bash
docker ps
```

**Terminal Output:**

```
[banner@stapp03 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Both checks return empty — a clean state on the destination server.

***

#### Step 9: Load the Image Archive

Use `docker load` to import the transferred archive. The image will be restored with its original name `games` and tag `datacenter`:

```bash
docker load -i /tmp/games.tar
```

**Terminal Output:**

```
[banner@stapp03 ~]$ docker load -i /tmp/games.tar
f2a7f0726353: Loading layer [==================================================>]  80.65MB/80.65MB
14ec25d93ab0: Loading layer [==================================================>]  60.77MB/60.77MB
Loaded image: games:datacenter
```

Two image layers were loaded — `80.65MB` and `60.77MB`. The final line `Loaded image: games:datacenter` confirms the image was restored with its exact original name and tag from App Server 1.

***

#### Step 10: Verify the Image on App Server 3

List all Docker images to confirm `games:datacenter` is now available:

```bash
docker images
```

**Terminal Output:**

```
[banner@stapp03 ~]$ docker images
REPOSITORY   TAG          IMAGE ID       CREATED          SIZE
games        datacenter   481b9a46c25a   15 minutes ago   139MB
[banner@stapp03 ~]$
```

The image `games:datacenter` is now present on App Server 3 with Image ID `481b9a46c25a`, size `139MB`, created `15 minutes ago` — matching the source image from App Server 1.

***

### Lab Complete

| Requirement               | Detail                                           | Status    |
| ------------------------- | ------------------------------------------------ | --------- |
| Source server             | `stapp01`                                        | Confirmed |
| Destination server        | `stapp03`                                        | Confirmed |
| Image archived            | `docker save -o /tmp/games.tar games:datacenter` | Confirmed |
| Archive size              | `135MB` (141,432,832 bytes)                      | Confirmed |
| Archive transferred       | `scp` at 107.9MB/s — 100% complete               | Confirmed |
| Docker service on stapp03 | `active (running)` since 03:28:18 UTC            | Confirmed |
| Image loaded              | `docker load -i /tmp/games.tar`                  | Confirmed |
| Image name on stapp03     | `games`                                          | Confirmed |
| Image tag on stapp03      | `datacenter`                                     | Confirmed |
| Image ID                  | `481b9a46c25a`                                   | Confirmed |

***

### Key Concepts

#### `docker save` vs `docker export`

Both commands produce `.tar` archives but they serve different purposes:

| Command         | Source    | Includes                                    | Use Case                      |
| --------------- | --------- | ------------------------------------------- | ----------------------------- |
| `docker save`   | Image     | All layers, history, metadata, tags         | Transfer images between hosts |
| `docker export` | Container | Filesystem snapshot only, no layers/history | Backup container filesystem   |

`docker save` was used in this lab because the requirement was to transfer the full image — including its tag `games:datacenter` — so it could be loaded identically on the destination server.

#### `docker load` vs `docker import`

The corresponding restore commands also differ:

| Command         | Restores                   | Preserves Tag | Use Case                          |
| --------------- | -------------------------- | ------------- | --------------------------------- |
| `docker load`   | Full image with all layers | Yes           | Restore a `docker save` archive   |
| `docker import` | Flat filesystem only       | No            | Restore a `docker export` archive |

`docker load` was used here because it preserves the original repository name and tag — which is why the image appeared as `games:datacenter` immediately after loading without any additional tagging step.

#### The Full Image Transfer Workflow

```
App Server 1 (stapp01)                    App Server 3 (stapp03)
┌─────────────────────┐                   ┌─────────────────────┐
│  games:datacenter   │                   │                     │
│  (Docker image)     │                   │  (no image yet)     │
│         |           │                   │                     │
│  docker save -o     │                   │                     │
│  /tmp/games.tar     │                   │                     │
│         |           │                   │                     │
│  games.tar (135MB)  │──── scp ─────────▶│  /tmp/games.tar     │
│                     │                   │         |           │
└─────────────────────┘                   │  docker load -i     │
                                          │  /tmp/games.tar     │
                                          │         |           │
                                          │  games:datacenter   │
                                          │  (481b9a46c25a)     │
                                          └─────────────────────┘
```

#### SCP for File Transfer Between Servers

`scp` (Secure Copy Protocol) uses SSH to securely transfer files between hosts. The syntax used in this lab:

```bash
scp /tmp/games.tar banner@stapp03:/tmp/
```

| Component        | Value            | Description                    |
| ---------------- | ---------------- | ------------------------------ |
| `/tmp/games.tar` | Source path      | File on the local server       |
| `banner`         | Remote user      | Username on destination server |
| `stapp03`        | Remote host      | Hostname of destination server |
| `/tmp/`          | Destination path | Directory on the remote server |

#### Why Image Size Differs Between Archive and Loaded Image

| State                  | Size                      |
| ---------------------- | ------------------------- |
| `.tar` archive on disk | 135MB (141,432,832 bytes) |
| Loaded image in Docker | 139MB                     |

The slight size difference is expected. The `.tar` archive stores compressed layer data, while the loaded image size reflects the uncompressed layer sizes as Docker stores them internally in the overlay2 storage driver.

***

_Lab completed on 2026-03-26 | Source: stapp01 | Destination: stapp03 | OS: CentOS Stream 9 | Docker: 26.1.3_



<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
