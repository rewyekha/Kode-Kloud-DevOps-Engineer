# Docker Update Permissions

One of the Nautilus project developers need access to run docker commands on `App Server 1`. This user is already created on the server. Accomplish this task as per details given below:

User `anita` is not able to run docker commands on `App Server 1` in Stratos DC, make the required changes so that this user can run docker commands without `sudo`.



## Linux Access Control: Grant Docker Command Access Without sudo

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Linux, Docker, User Management, Group Permissions

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 1](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-1)
   * [Step 2: Switch to Root](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-switch-to-root)
   * [Step 3: Add User to Docker Group](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-add-user-to-docker-group)
   * [Step 4: Verify Group Membership](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-group-membership)
   * [Step 5: Test Docker Access as anita](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-test-docker-access-as-anita)
4. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

One of the Nautilus project developers needs access to run docker commands on **App Server 1**. This user is already created on the server. Accomplish this task as per the details given below:

User `anita` is not able to run docker commands on **App Server 1** in Stratos DC. Make the required changes so that this user can run docker commands **without** `sudo`.

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

> **Target:** Application Server 1 (`stapp01`) — add user `anita` to the `docker` group.

***

### Solution

#### Step 1: SSH into Application Server 1

From the jump host, connect to `stapp01` as user `tony`:

```bash
ssh tony@stapp01
```

**Terminal Output:**

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.97.249)' can't be established.
ED25519 key fingerprint is SHA256:guZxI15+d8pk74VCoR/V4EcJnNq/Y90/HbuoYNUs1Zc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 2: Switch to Root

```bash
sudo su -
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ sudo su -
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
[root@stapp01 ~]#
```

Enter password: `Ir0nM@n`

***

#### Step 3: Add User to Docker Group

Add `anita` to the `docker` group using `usermod`. The `-aG` flags mean **append** to **supplementary groups** — this adds the user to the docker group without removing them from any existing groups:

```bash
usermod -aG docker anita
```

**Terminal Output:**

```bash
[root@stapp01 ~]# usermod -aG docker anita
[root@stapp01 ~]#
```

No output means the command succeeded.

***

#### Step 4: Verify Group Membership

Confirm `anita` now belongs to the `docker` group using two methods:

**Method 1 — Check user identity and all group memberships:**

```bash
id anita
```

**Terminal Output:**

```bash
[root@stapp01 ~]# id anita
uid=1001(anita) gid=1001(anita) groups=1001(anita),992(docker)
```

`docker` group with GID `992` is now listed in `anita`'s groups.

**Method 2 — Check the docker group entry directly:**

```bash
grep docker /etc/group
```

**Terminal Output:**

```bash
[root@stapp01 ~]# grep docker /etc/group
docker:x:992:tony,anita
```

Both `tony` and `anita` are now members of the `docker` group.

***

#### Step 5: Test Docker Access as anita

Switch to `anita` and run docker commands to confirm access works without `sudo`:

```bash
su - anita
```

**Test 1 — List running containers:**

```bash
docker ps
```

**Terminal Output:**

```bash
[anita@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Clean output with no permission errors — `anita` can query the Docker daemon.

**Test 2 — Full Docker engine info:**

```bash
docker info
```

**Terminal Output:**

```bash
[anita@stapp01 ~]$ docker info
Client: Docker Engine - Community
 Version:    26.1.3
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.31.1
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.0.2
    Path:     /usr/libexec/docker/cli-plugins/docker-compose
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 26.1.3
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: false
  userxattr: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 8b3b7ca2e5ce38e8f31a34f35b2b68ceb8470d89
 runc version: v1.1.12-0-g51d5e94
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.8.0-90-generic
 Operating System: CentOS Stream 9
 OSType: linux
 Architecture: x86_64
 CPUs: 16
 Total Memory: 61.92GiB
 Name: stapp01
 ID: b49e6fe9-038e-4e13-8087-f49985d0c2fc
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  docker-registry-mirror.kodekloud.com
  127.0.0.0/8
 Registry Mirrors:
  http://docker-registry-mirror.kodekloud.com/
 Live Restore Enabled: false
[anita@stapp01 ~]$
```

`docker info` returned full Docker Engine details — Server Version `26.1.3` running on CentOS Stream 9 — with no permission denied errors.

***

### Lab Complete

| Task                       | Detail                     | Status    |
| -------------------------- | -------------------------- | --------- |
| Target server              | `stapp01`                  | Confirmed |
| Target user                | `anita`                    | Confirmed |
| Group added                | `docker` (GID 992)         | Confirmed |
| Command used               | `usermod -aG docker anita` | Executed  |
| `docker ps` without sudo   | Clean output               | Confirmed |
| `docker info` without sudo | Full engine info returned  | Confirmed |

***

### Key Concepts

#### How Docker Permissions Work

The Docker CLI communicates with the Docker daemon via a Unix socket located at `/var/run/docker.sock`. By default, this socket is owned by the `root` user and the `docker` group:

```bash
ls -la /var/run/docker.sock
srw-rw---- 1 root docker 0 Mar 24 05:00 /var/run/docker.sock
```

Any user that belongs to the `docker` group has read/write access to this socket — which grants full control of the Docker daemon without needing `sudo`.

#### The `usermod -aG` Command

```bash
usermod -aG docker anita
```

| Flag | Meaning                                                                         |
| ---- | ------------------------------------------------------------------------------- |
| `-a` | Append — adds the user to the group without removing existing group memberships |
| `-G` | Supplementary groups — specifies the group to add the user to                   |

> **Warning:** Using `-G` without `-a` would replace all of the user's supplementary groups with only the specified group, potentially removing them from important groups like `wheel` or `sudo`.

#### Why a New Login Session is Required

Group membership changes take effect only for **new login sessions**. In this lab, switching to `anita` with `su - anita` starts a new session that picks up the updated group membership immediately. If `anita` was already logged in before the change, she would need to log out and back in for it to take effect.

#### Security Consideration

Adding a user to the `docker` group is functionally equivalent to granting them `root`-level access on the host. A user with docker group membership can:

* Mount the host filesystem into a container
* Run privileged containers
* Escape container isolation

This access should only be granted to trusted developers and administrators who genuinely require it.

***

_Lab completed on 2026-03-24 | Server: stapp01 | OS: CentOS Stream 9 | Docker: 26.1.3_

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
