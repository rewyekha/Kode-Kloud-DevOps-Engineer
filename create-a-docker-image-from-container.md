# Create a Docker Image From Container

One of the Nautilus developer was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container. Below are more details about it:

a. Create an image `apps:nautilus` on `Application Server 3` from a container `ubuntu_latest` that is running on same server.



## Docker: Create an Image From a Running Container

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker, Images, Containers, docker commit

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-3)
   * [Step 2: Switch to Root](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-switch-to-root)
   * [Step 3: Verify the Running Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-verify-the-running-container)
   * [Step 4: Check Existing Images](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-check-existing-images)
   * [Step 5: Create the Image From the Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-the-image-from-the-container)
   * [Step 6: Verify the Image Was Created](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-verify-the-image-was-created)
4. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

One of the Nautilus developers was working to test new changes on a container. He wants to keep a backup of his changes to the container. A new request has been raised for the DevOps team to create a new image from this container.

**Requirements:**

* Create an image `apps:nautilus` on **Application Server 3** from a container named `ubuntu_latest` that is running on the same server.

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

> **Target:** Application Server 3 (`stapp03`) — create image from container `ubuntu_latest`.

***

### Solution

#### Step 1: SSH into Application Server 3

From the jump host, connect to `stapp03` as user `banner`:

```bash
ssh banner@stapp03
```

**Terminal Output:**

```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.234.211)' can't be established.
ED25519 key fingerprint is SHA256:oqpmN8d8m51lu80IOmSM8UpaAoxs+c7IwXZARyoz9io.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
[banner@stapp03 ~]$
```

Enter password: `BigGr33n`

***

#### Step 2: Switch to Root

```bash
sudo su -
```

**Terminal Output:**

```bash
[banner@stapp03 ~]$ sudo su -
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
[root@stapp03 ~]#
```

Enter password: `BigGr33n`

***

#### Step 3: Verify the Running Container

Confirm the container `ubuntu_latest` is present and in a running state:

```bash
docker ps
```

**Terminal Output:**

```bash
[root@stapp03 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
bd323ef28db5   ubuntu    "/bin/bash"   About a minute ago   Up About a minute             ubuntu_latest
```

The container `ubuntu_latest` is confirmed running with:

| Field        | Value               |
| ------------ | ------------------- |
| Container ID | `bd323ef28db5`      |
| Base Image   | `ubuntu`            |
| Command      | `/bin/bash`         |
| Status       | `Up About a minute` |
| Name         | `ubuntu_latest`     |

***

#### Step 4: Check Existing Images

Before committing, check what images are currently on the server:

```bash
docker images
```

**Terminal Output:**

```bash
[root@stapp03 ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    f794f40ddfff   4 weeks ago   78.1MB
```

Only the base `ubuntu:latest` image exists at this point. The `apps:nautilus` image does not yet exist.

***

#### Step 5: Create the Image From the Container

Use `docker commit` to snapshot the current state of the running container into a new image named `apps:nautilus`:

```bash
docker commit ubuntu_latest apps:nautilus
```

**Terminal Output:**

```bash
[root@stapp03 ~]# docker commit ubuntu_latest apps:nautilus
sha256:edbdcf14fd260c64a88130aa581f8b763759d0b04827c575f1a57ee6f79d851d
```

Docker returns the full SHA256 digest of the newly created image: `edbdcf14fd260c64a88130aa581f8b763759d0b04827c575f1a57ee6f79d851d`

***

#### Step 6: Verify the Image Was Created

List all images to confirm `apps:nautilus` is now present:

```bash
docker images
```

**Terminal Output:**

```bash
[root@stapp03 ~]# docker images
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
apps         nautilus   edbdcf14fd26   6 seconds ago   138MB
ubuntu       latest     f794f40ddfff   4 weeks ago     78.1MB
[root@stapp03 ~]#
```

The image `apps:nautilus` has been successfully created.

| Field      | Value          |
| ---------- | -------------- |
| Repository | `apps`         |
| Tag        | `nautilus`     |
| Image ID   | `edbdcf14fd26` |
| Created    | 6 seconds ago  |
| Size       | `138MB`        |

***

### Lab Complete

| Task             | Detail                                      | Status    |
| ---------------- | ------------------------------------------- | --------- |
| Target server    | `stapp03`                                   | Confirmed |
| Source container | `ubuntu_latest` (ID: `bd323ef28db5`)        | Running   |
| New image name   | `apps:nautilus`                             | Created   |
| Image ID         | `edbdcf14fd26`                              | Confirmed |
| Image size       | `138MB`                                     | Confirmed |
| Command used     | `docker commit ubuntu_latest apps:nautilus` | Executed  |

***

### Key Concepts

#### What is `docker commit`?

`docker commit` creates a new Docker image from the **current filesystem state** of a running or stopped container. It captures all file changes, new files, and modifications made inside the container since it was started — essentially taking a snapshot of the container's writable layer and persisting it as a new read-only image layer.

```bash
Container (ubuntu_latest)
    └── Writable layer (developer changes)
            |
            | docker commit
            v
New Image (apps:nautilus)
    └── Read-only layer (developer changes frozen)
    └── Base layer (ubuntu:latest)
```

#### Image Size Difference Explained

```bash
ubuntu:latest    78.1MB   (base image — no changes)
apps:nautilus   138.0MB   (base image + developer changes)
                --------
Difference      ~60MB     (represents all changes made inside the container)
```

The additional 60MB represents all filesystem modifications the developer made inside the `ubuntu_latest` container — installed packages, created files, configuration changes — all of which are now preserved in the new image.

#### Image Naming Convention: `repository:tag`

```bash
apps:nautilus
|     |
|     └── Tag — version or variant label (e.g. nautilus, latest, v1.0)
└── Repository — the image name (e.g. apps, ubuntu, nginx)
```

When a tag is not specified, Docker defaults to `latest`. In this lab, the tag `nautilus` explicitly identifies this as the Nautilus project variant of the `apps` image.

#### `docker commit` vs Dockerfile

| Method          | Use Case                                                                                                       |
| --------------- | -------------------------------------------------------------------------------------------------------------- |
| `docker commit` | Quick snapshot of manual changes made in a running container. Good for backups and capturing exploratory work. |
| `Dockerfile`    | Reproducible, version-controlled, automated image builds. Preferred for production workflows.                  |

`docker commit` is useful for preserving the state of interactive work, but the resulting image is not reproducible from source — the exact commands that produced the state are not recorded. For production images, a Dockerfile is always recommended.

#### The SHA256 Digest

When `docker commit` succeeds, it outputs a full SHA256 digest:

```bash
sha256:edbdcf14fd260c64a88130aa581f8b763759d0b04827c575f1a57ee6f79d851d
```

This is a cryptographic hash of the image's content. Docker uses this hash internally to uniquely identify every image layer. The first 12 characters (`edbdcf14fd26`) are displayed as the short Image ID in `docker images` output.

***

_Lab completed on 2026-03-24 | Server: stapp03 | OS: CentOS Stream 9 | Docker: 26.1.3_





<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
