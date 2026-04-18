# Day 35: Install Docker Packages and Start Docker Service

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The Nautilus DevOps team aims to containerize various applications following a recent meeting with the application development team. They intend to conduct testing with the following steps:

1. Install `docker-ce` and `docker compose` packages on `App Server 1`.
2. Initiate the `docker` service.

**Environment:** Nautilus DevOps Lab\
**Target Server:** `stapp01` (App Server 1)\
**OS:** CentOS Stream 9

***

### Problem Statement

Install the following on **App Server 1 (stapp01)**:

* `docker-ce`
* `docker-compose` (plugin)
* Start and enable Docker service

***

## Step 1: Login to App Server

```bash
thor@jumphost ~$ ssh tony@stapp01
```

#### First Time SSH Warning

```bash
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:YAcKCLyjeUtFq4AUW8z8JItbiyFNd4k2hg1U5CysoYU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
```

Enter password:

```
Ir0nM@n
```

***

## Step 2: Switch to Root User

```bash
[tony@stapp01 ~]$ sudo su -
```

Output:

```bash
We trust you have received the usual lecture from the local System Administrator.
[sudo] password for tony:
```

***

## Step 3: Initial Docker Installation Attempt (Fails)

```bash
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### Error Output

```bash
No match for argument: docker-ce
No match for argument: docker-ce-cli
No match for argument: containerd.io
No match for argument: docker-compose-plugin
Error: Unable to find a match
```

***

### Why This Error Occurs?

CentOS Stream 9 **does not include Docker CE in default repositories**.

Docker official repository must be added manually.

***

## Step 4: Trying to Start Docker (Fails)

```bash
systemctl enable docker
```

Output:

```bash
Failed to enable unit: Unit file docker.service does not exist.
```

```bash
systemctl start docker
```

Output:

```bash
Failed to start docker.service: Unit docker.service not found.
```

- Reason: Docker is not installed yet.

***

## Step 5: Install Required Utility

```bash
dnf install -y yum-utils
```

#### Successful Output Summary

```bash
Installed:
  yum-utils-4.3.0-26.el9.noarch
Complete!
```

***

## Step 6: Add Docker Official Repository

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Output:

```bash
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
```

Verify:

```bash
dnf repolist | grep docker
```

Output:

```bash
docker-ce-stable    Docker CE Stable - x86_64
```

***

## Step 7: Install Docker Packages (Successful)

```bash
dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### Key Installation Summary

```bash
Installed:
  docker-ce-29.2.1
  docker-ce-cli-29.2.1
  docker-compose-plugin-5.1.0
  containerd.io-2.2.1
Complete!
```

***

## GPG Key Import (Normal Behavior)

```bash
Importing GPG key 0x621E9F35:
Userid     : "Docker Release (CE rpm) <docker@docker.com>"
Key imported successfully
```

- This is expected during first installation.

***

## Step 8: Start and Enable Docker

```bash
systemctl start docker
systemctl enable docker
```

Output:

```bash
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
```

***

## Step 9: Verify Docker Status

```bash
systemctl status docker
```

#### Successful Output

```bash
Active: active (running)
```

Full Service Info:

```bash
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled)
   Active: active (running)
```

***

## Warning Messages Seen (Non-Critical)

```bash
Failed to set 'blkio.weight' attribute...
Failed to reset devices.allow/devices.deny...
Operation not permitted
```

#### Should You Worry?

- No.\
These are **cgroup permission warnings** common in lab/virtualized environments.\
Docker is running properly.

***

## Step 10: Verify Docker & Compose Versions

```bash
docker --version
docker compose version
```

Output:

```bash
Docker version 29.2.1, build a5c7197
Docker Compose version v5.1.0
```

***

## Final Verification Checklist

| Check | Status |
| ---------------------- | ------ |
| Docker Repo Added | |
| Docker Installed | |
| Docker Service Started | |
| Docker Enabled on Boot | |
| Docker Running | |
| Docker Compose Working | |

***

## Final Result

Docker CE and Docker Compose successfully installed and running on:

**Server:** `stapp01.stratos.xfusioncorp.com`\
**Environment:** Nautilus DevOps Lab

***

## Common Errors You May Face

| Error | Reason | Fix |
| --------------------------------- | -------------------- | ----------------- |
| `No match for argument docker-ce` | Repo not added | Add Docker repo |
| `Unit docker.service not found` | Docker not installed | Install docker-ce |
| `Permission denied` | Not root | Use `sudo su -` |
| GPG Key prompt | First-time install | Accept import |

***

- **Day 35 Task Completed Successfully**

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
