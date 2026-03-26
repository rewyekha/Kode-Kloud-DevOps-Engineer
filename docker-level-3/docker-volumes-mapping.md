# Docker Volumes Mapping

The Nautilus DevOps team is testing applications containerization, which is supposed to be migrated on docker container-based environments soon. In today's stand-up meeting one of the team members has been assigned a task to create and test a docker container with certain requirements. Below are more details:

a. On `App Server 3` in `Stratos DC` pull `nginx` image (preferably `latest` tag but others should work too).

b. Create a new container with name `news` from the image you just pulled.

c. Map the host volume `/opt/finance` with container volume `/home`. There is an `sample.txt` file present on same server under `/tmp`; copy that file to `/opt/finance`. Also please keep the container in running state.



## Docker: Create a Container with Volume Mapping and File Copy

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker, Containers, Volume Mounting, nginx

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 3](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-3)
   * [Step 2: Switch to Root](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-switch-to-root)
   * [Step 3: Pull the nginx Image](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-pull-the-nginx-image)
   * [Step 4: Create the Container with Volume Mapping](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-create-the-container-with-volume-mapping)
   * [Step 5: Verify Container Was Created](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-verify-container-was-created)
   * [Step 6: Start the Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-start-the-container)
   * [Step 7: Locate and Copy the Sample File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-locate-and-copy-the-sample-file)
   * [Step 8: Verify the Container is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-verify-the-container-is-running)
   * [Step 9: Verify File is Visible Inside the Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-9-verify-file-is-visible-inside-the-container)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Alternative Approach](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#alternative-approach)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus DevOps team is testing application containerization, which is supposed to be migrated to docker container-based environments soon. A team member has been assigned a task to create and test a docker container with certain requirements.

**Requirements:**

* **a.** On **App Server 3** in Stratos DC, pull the `nginx` image (preferably `latest` tag but others should work too).
* **b.** Create a new container named `news` from the image you just pulled.
* **c.** Map the host volume `/opt/finance` with container volume `/home`. There is a `sample.txt` file present on the same server under `/tmp` — copy that file to `/opt/finance`. Keep the container in a running state.

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

> **Target:** Application Server 3 (`stapp03`) — all commands are executed on this server.

***

### Solution

#### Step 1: SSH into Application Server 3

From the jump host, connect to `stapp03` as user `banner`:

```bash
ssh banner@stapp03
```

**Terminal Output:**

```
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.164.7)' can't be established.
ED25519 key fingerprint is SHA256:xij+uLMp3mj/WqTfCWw7dLgslnVEMltBILaQuyRzY3I.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
[banner@stapp03 ~]$
```

Enter password: `BigGr33n`

***

#### Step 2: Switch to Root

Docker commands require elevated privileges on this server. Switch to root using `sudo su`:

```bash
sudo su
```

**Terminal Output:**

```
[banner@stapp03 ~]$ sudo su
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
[root@stapp03 banner]#
```

Enter password: `BigGr33n`

***

#### Step 3: Pull the nginx Image

Pull the `nginx:latest` image from Docker Hub. This downloads all the required image layers to the local Docker image cache:

```bash
docker pull nginx:latest
```

**Terminal Output:**

```
[root@stapp03 banner]# docker pull nginx:latest
latest: Pulling from library/nginx
ec781dee3f47: Pull complete
bb3d0aa29654: Pull complete
510ddf6557d6: Pull complete
cde7a05ae428: Pull complete
587e3d84dbb5: Pull complete
3189680c601f: Pull complete
5e815e07e569: Pull complete
Digest: sha256:7150b3a39203cb5bee612ff4a9d18774f8c7caf6399d6e8985e97e28eb751c18
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

Seven layers were downloaded and the image was stored locally. The digest `sha256:7150b3a3...` uniquely identifies this version of the image.

***

#### Step 4: Create the Container with Volume Mapping

Create the container named `news` with the volume mapping `/opt/finance` on the host mounted to `/home` inside the container. The `docker create` command creates the container but does not start it yet:

```bash
docker create -v /opt/finance:/home --name news nginx:latest
```

**Terminal Output:**

```
[root@stapp03 banner]# docker create -v /opt/finance:/home --name news nginx:latest
04718f97b96a6bcb2b812bd2b8bcc1edaad5136a7a9d631781a1a796170d7832
```

Docker returns the full container ID `04718f97b96a6bcb2b812bd2b8bcc1edaad5136a7a9d631781a1a796170d7832`, confirming the container was created successfully.

The volume flag `-v /opt/finance:/home` means:

* Left side `/opt/finance` — directory on the **host** server
* Right side `/home` — directory inside the **container**

Both directories are kept in sync as a live mount.

***

#### Step 5: Verify Container Was Created

Check the container exists in a `Created` state (not yet running):

```bash
docker ps -a
```

**Terminal Output:**

```
[root@stapp03 banner]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS    PORTS     NAMES
04718f97b96a   nginx:latest   "/docker-entrypoint.…"   5 seconds ago   Created             news
```

The container `news` is listed with status `Created`. The `-a` flag shows all containers including those not yet running. Without `-a`, a `Created` container would not appear in `docker ps` output.

***

#### Step 6: Start the Container

Start the `news` container to put it into a running state:

```bash
docker start news
```

**Terminal Output:**

```
[root@stapp03 banner]# docker start news
news
```

Docker prints the container name `news` to confirm it was started successfully.

***

#### Step 7: Locate and Copy the Sample File

Check that `sample.txt` exists in `/tmp` as stated in the lab requirements, then copy it to the host volume directory `/opt/finance`:

```bash
ls /tmp
```

**Terminal Output:**

```
[root@stapp03 banner]# ls /tmp
sample.txt
systemd-private-c3e880158dfe481c897ec6163084711f-dbus-broker.service-CcR5yp
systemd-private-c3e880158dfe481c897ec6163084711f-rtkit-daemon.service-Cd1OFh
systemd-private-c3e880158dfe481c897ec6163084711f-systemd-logind.service-CI5qWk
systemd-private-c3e880158dfe481c897ec6163084711f-upower.service-2Qpae1
```

`sample.txt` is confirmed present in `/tmp`. Copy it to the host volume:

```bash
cp /tmp/sample.txt /opt/finance/
```

**Terminal Output:**

```
[root@stapp03 banner]# cp /tmp/sample.txt /opt/finance/
[root@stapp03 banner]#
```

No output means the copy succeeded. Because `/opt/finance` is live-mounted to `/home` inside the container, the file becomes immediately accessible inside the container without any restart or additional steps.

***

#### Step 8: Verify the Container is Running

Confirm the `news` container is in `Up` status:

```bash
docker ps
```

**Terminal Output:**

```
[root@stapp03 banner]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
04718f97b96a   nginx:latest   "/docker-entrypoint.…"   3 minutes ago   Up 2 minutes   80/tcp    news
```

The container `news` is `Up 2 minutes`, running on port `80/tcp`, using `nginx:latest`. All requirements for a running container are satisfied.

***

#### Step 9: Verify File is Visible Inside the Container

Execute a command inside the running container to confirm the `sample.txt` file is accessible at `/home`:

```bash
docker exec -it news ls /home
```

**Terminal Output:**

```
[root@stapp03 banner]# docker exec -it news ls /home
sample.txt
[root@stapp03 banner]#
```

`sample.txt` is visible inside the container at `/home`. This confirms the volume mount is working correctly — the file was written to `/opt/finance` on the host and is immediately reflected inside the container at `/home`.

***

### Lab Complete

| Requirement               | Detail                                         | Status    |
| ------------------------- | ---------------------------------------------- | --------- |
| Server                    | `stapp03`                                      | Confirmed |
| Image pulled              | `nginx:latest` (digest: `sha256:7150b3a3...`)  | Confirmed |
| Container name            | `news`                                         | Confirmed |
| Volume mapping            | `/opt/finance` → `/home`                       | Confirmed |
| Sample file copied        | `/tmp/sample.txt` → `/opt/finance/sample.txt`  | Confirmed |
| File visible in container | `docker exec news ls /home` shows `sample.txt` | Confirmed |
| Container state           | `Up 2 minutes` (running)                       | Confirmed |

***

### Alternative Approach

The lab was completed using `docker create` followed by `docker start`. An equivalent single-command approach using `docker run` also achieves the same result:

```bash
docker run -d --name news -v /opt/finance:/home nginx:latest
```

Then copy the file as before:

```bash
cp /tmp/sample.txt /opt/finance/
```

| Approach           | Commands   | Result                                    |
| ------------------ | ---------- | ----------------------------------------- |
| `create` + `start` | 2 commands | Container created then started separately |
| `docker run -d`    | 1 command  | Container created and started in one step |

Both approaches produce an identical running container. The `create` + `start` pattern is useful when you need to inspect or modify a container before it begins running. The `run -d` pattern is more concise for straightforward use cases.

***

### Key Concepts

#### Docker Volume Mounting

Docker volumes connect a directory on the host filesystem to a directory inside a container. The syntax is:

```
-v <host_path>:<container_path>
```

```
Host Server (stapp03)          Container (news)
/opt/finance/                  /home/
    sample.txt    ←——live——→       sample.txt
```

The mount is **live and bidirectional** — any change on either side is instantly reflected on the other. This is why copying `sample.txt` to `/opt/finance` after the container started still made the file visible inside the container at `/home` without any restart.

#### `docker create` vs `docker run`

| Command         | Creates Container | Starts Container | Use Case                                  |
| --------------- | ----------------- | ---------------- | ----------------------------------------- |
| `docker create` | Yes               | No               | Pre-configure a container before starting |
| `docker start`  | No                | Yes              | Start a previously created container      |
| `docker run`    | Yes               | Yes              | Create and start in a single step         |

#### `docker ps` vs `docker ps -a`

| Command        | Shows                                                 |
| -------------- | ----------------------------------------------------- |
| `docker ps`    | Only **running** containers                           |
| `docker ps -a` | **All** containers including Created, Stopped, Exited |

A container in `Created` state is not visible in plain `docker ps` output — the `-a` flag is required to see it before it is started.

#### Why Copying After Start Still Works

A common misconception is that files must exist in the host volume directory before the container starts. In reality, Docker volume mounts are live at the filesystem level. The mount point `/opt/finance` on the host is continuously synced with `/home` inside the container. Any file added to `/opt/finance` at any time — before or after the container starts — is immediately visible inside the container. No restart is required.

#### `docker exec` for In-Container Verification

```bash
docker exec -it news ls /home
```

| Flag       | Meaning                             |
| ---------- | ----------------------------------- |
| `-i`       | Interactive — keeps stdin open      |
| `-t`       | Allocate a pseudo-TTY terminal      |
| `news`     | Target container name               |
| `ls /home` | Command to run inside the container |

`docker exec` runs a command inside an already running container without interrupting its main process. It is the standard tool for inspecting container state, verifying file presence, and debugging running containers.

***

_Lab completed on 2026-03-25 | Server: stapp03 | OS: CentOS Stream 9 | Docker: 26.1.3_

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
