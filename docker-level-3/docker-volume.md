# Page 1

The Nautilus application development team shared static website content that needs to be hosted on the `httpd` web server using a containerised platform. The team has shared details with the DevOps team, and we need to set up an environment according to those guidelines. Below are the details:

a. On `App Server 1` in `Stratos DC` create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (please use the exact name for file).

b. Use `httpd` (preferably `latest` tag) image for container and make sure container is named as `httpd`; you can use any name for service.

c. Map `80` number port of container with port `6400` of docker host.

d. Map container's `/usr/local/apache2/htdocs` volume with `/opt/data` volume of docker host which is already there. (please do not modify any data within these locations).



## Docker Compose: Deploy httpd Web Server with Volume and Port Mapping

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker Compose, httpd, Volume Mapping, Port Mapping, Containerisation

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 1](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-1)
   * [Step 2: Verify Docker Installation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-verify-docker-installation)
   * [Step 3: Install Docker Compose Plugin](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-install-docker-compose-plugin)
   * [Step 4: Verify Docker Compose Version](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-verify-docker-compose-version)
   * [Step 5: Create the Docker Directory](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-create-the-docker-directory)
   * [Step 6: Write the Docker Compose File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-write-the-docker-compose-file)
   * [Step 7: Review the Compose File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-review-the-compose-file)
   * [Step 8: Start the Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-start-the-container)
   * [Step 9: Verify the Container is Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-9-verify-the-container-is-running)
   * [Step 10: Test the Web Server](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-10-test-the-web-server)
4. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
5. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus application development team shared static website content that needs to be hosted on the `httpd` web server using a containerised platform. The team has shared details with the DevOps team, and an environment needs to be set up according to those guidelines.

**Requirements:**

* **a.** On **App Server 1** in Stratos DC create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (use the exact name for the file).
* **b.** Use `httpd` (preferably `latest` tag) image for the container and make sure container is named as `httpd`. Any name can be used for the service.
* **c.** Map port `80` of the container with port `6400` of the docker host.
* **d.** Map container's `/usr/local/apache2/htdocs` volume with `/opt/data` volume of docker host which is already there. Do not modify any data within these locations.

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

> **Target:** Application Server 1 (`stapp01`) — all work is performed on this server.

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
The authenticity of host 'stapp01 (10.244.164.27)' can't be established.
ED25519 key fingerprint is SHA256:N8tDvzKdUj6po/nJxdxvLyzRCCLbb7IRM9BWndldaQM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 2: Verify Docker Installation

Check the `/opt/docker` directory and confirm Docker is installed:

```bash
ls /opt/docker/
docker --version
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ ls /opt/docker/
[tony@stapp01 ~]$ docker --version
Docker version 26.1.3, build b72abbb
```

The `/opt/docker/` directory exists but is empty — it will be used for the compose file. Docker `26.1.3` is installed and available.

***

#### Step 3: Install Docker Compose Plugin

Check whether `docker-compose` is available as a standalone binary:

```bash
docker-compose --version
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ docker-compose --version
-bash: docker-compose: command not found
```

The standalone `docker-compose` binary is not installed. The modern `docker compose` plugin must be installed via `yum`. An initial attempt with `apt` was made, which also confirmed this is a CentOS/RHEL system:

```bash
sudo apt update
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ sudo apt update
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
sudo: apt: command not found
```

`apt` is not available — the server is CentOS Stream 9. Install using `yum`:

```bash
sudo yum install docker-compose-plugin -y
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ sudo yum install docker-compose-plugin -y
Last metadata expiration check: 0:00:45 ago on Thu Mar 26 04:18:28 2026.
Package docker-compose-plugin-5.0.2-1.el9.x86_64 is already installed.
Dependencies resolved.
================================================================================================================================================
 Package                                  Architecture              Version                           Repository                           Size
================================================================================================================================================
Upgrading:
 docker-compose-plugin                    x86_64                    5.1.1-1.el9                       docker-ce-stable                    8.2 M

Transaction Summary
================================================================================================================================================
Upgrade  1 Package

Total download size: 8.2 M
Downloading Packages:
docker-compose-plugin-5.1.1-1.el9.x86_64.rpm                                             72 MB/s | 8.2 MB     00:00
------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                      71 MB/s | 8.2 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                        1/1
  Upgrading        : docker-compose-plugin-5.1.1-1.el9.x86_64                                                                             1/2
  Running scriptlet: docker-compose-plugin-5.1.1-1.el9.x86_64                                                                             1/2
  Running scriptlet: docker-compose-plugin-5.0.2-1.el9.x86_64                                                                             2/2
  Cleanup          : docker-compose-plugin-5.0.2-1.el9.x86_64                                                                             2/2
  Running scriptlet: docker-compose-plugin-5.0.2-1.el9.x86_64                                                                             2/2
  Verifying        : docker-compose-plugin-5.1.1-1.el9.x86_64                                                                             1/2
  Verifying        : docker-compose-plugin-5.0.2-1.el9.x86_64                                                                             2/2

Upgraded:
  docker-compose-plugin-5.1.1-1.el9.x86_64

Complete!
```

The existing `docker-compose-plugin-5.0.2` was upgraded to `5.1.1` from the `docker-ce-stable` repository. The upgrade downloaded `8.2MB` at `72MB/s`.

***

#### Step 4: Verify Docker Compose Version

Confirm `docker compose` is now available as a plugin subcommand:

```bash
docker compose version
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ docker compose version
Docker Compose version v5.1.1
```

`docker compose` is available as `v5.1.1`. Note: the modern plugin syntax is `docker compose` (with a space), not the legacy `docker-compose` (with a hyphen).

***

#### Step 5: Create the Docker Directory

Create the `/opt/docker` directory where the compose file will be stored, and navigate into it:

```bash
sudo mkdir -p /opt/docker
cd /opt/docker
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ sudo mkdir -p /opt/docker
[tony@stapp01 ~]$ cd /opt/docker
[tony@stapp01 docker]$
```

The `-p` flag ensures no error is thrown if the directory already exists.

***

#### Step 6: Write the Docker Compose File

Create the compose file at the exact path `/opt/docker/docker-compose.yml` using `nano`:

```bash
sudo nano /opt/docker/docker-compose.yml
```

**File content written:**

```yaml
version: '3.8'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6400:80"
    volumes:
      - /opt/data:/usr/local/apache2/htdocs
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ sudo nano /opt/docker/docker-compose.yml
[tony@stapp01 docker]$
```

Save and exit: `Ctrl+O` → `Enter` → `Ctrl+X`

***

#### Step 7: Review the Compose File

Verify the compose file contents are correct before deploying:

```bash
cat /opt/docker/docker-compose.yml
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ cat /opt/docker/docker-compose.yml
version: '3.8'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "6400:80"
    volumes:
      - /opt/data:/usr/local/apache2/htdocs
```

All four requirements are satisfied in the compose file:

| Requirement    | Configuration                                    |
| -------------- | ------------------------------------------------ |
| Container name | `container_name: httpd`                          |
| Image          | `image: httpd:latest`                            |
| Port mapping   | `"6400:80"` — host port 6400 → container port 80 |
| Volume mapping | `/opt/data:/usr/local/apache2/htdocs`            |

***

#### Step 8: Start the Container

Launch the container in detached mode using `docker compose up -d`:

```bash
docker compose up -d
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ docker compose up -d
WARN[0000] /opt/docker/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] up 9/9
 ✔ Image httpd:latest     Pulled                                                                                                            3.5s
 ✔ Network docker_default Created                                                                                                           0.1s
 ✔ Container httpd        Started                                                                                                           0.5s
```

Three actions were performed by Docker Compose:

1. `httpd:latest` image was pulled from Docker Hub in `3.5s`
2. A default network `docker_default` was created in `0.1s`
3. The container `httpd` was started in `0.5s`

> The warning `the attribute 'version' is obsolete` is informational only — Docker Compose v2 no longer requires the `version` key. The warning does not affect functionality and can be ignored or resolved by removing the `version: '3.8'` line.

***

#### Step 9: Verify the Container is Running

Confirm the `httpd` container is running with the correct port mapping:

```bash
docker ps
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ docker ps
CONTAINER ID   IMAGE          COMMAND              CREATED         STATUS         PORTS                                   NAMES
04c56adb2c6b   httpd:latest   "httpd-foreground"   9 seconds ago   Up 8 seconds   0.0.0.0:6400->80/tcp, :::6400->80/tcp   httpd
```

The container `httpd` is `Up 8 seconds` with port mapping `0.0.0.0:6400->80/tcp` confirmed — both IPv4 and IPv6 (`:::6400->80/tcp`) are listening. The container is running the `httpd-foreground` command as expected.

***

#### Step 10: Test the Web Server

Send a `curl` request to `localhost:6400` to confirm the Apache web server is serving content from the mounted volume:

```bash
curl http://localhost:6400
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ curl http://localhost:6400
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
 <head>
  <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
<ul><li><a href="index1.html"> index1.html</a></li>
</ul>
</body></html>
[tony@stapp01 docker]$
```

The Apache HTTP server responded with an `Index of /` page listing `index1.html`, which is the static content stored in `/opt/data` on the host — confirming the volume mount is working correctly and the web server is serving the pre-existing content without modification.

***

### Lab Complete

| Requirement         | Configuration                    | Status    |
| ------------------- | -------------------------------- | --------- |
| Server              | `stapp01`                        | Confirmed |
| Compose file path   | `/opt/docker/docker-compose.yml` | Confirmed |
| Container name      | `httpd`                          | Confirmed |
| Image               | `httpd:latest`                   | Confirmed |
| Host port           | `6400`                           | Confirmed |
| Container port      | `80`                             | Confirmed |
| Host volume         | `/opt/data`                      | Confirmed |
| Container volume    | `/usr/local/apache2/htdocs`      | Confirmed |
| Container status    | `Up 8 seconds`                   | Confirmed |
| Web server response | `Index of /` with `index1.html`  | Confirmed |

***

### Key Concepts

#### Docker Compose File Structure

The compose file used in this lab follows the standard service definition pattern:

```yaml
services:
  <service-name>:          # logical name (can be anything)
    image:                 # Docker image to use
    container_name:        # explicit container name (overrides auto-generated name)
    ports:                 # port mapping: "host:container"
      - "6400:80"
    volumes:               # volume mapping: "host_path:container_path"
      - /opt/data:/usr/local/apache2/htdocs
```

#### Port Mapping Syntax

```bash
"6400:80"
  |    |
  |    └── Container port — the port the application listens on inside the container
  └──────── Host port — the port exposed on the Docker host machine
```

External traffic hitting the host on port `6400` is forwarded by Docker to port `80` inside the container where Apache is listening.

#### Volume Mapping Syntax

```bash
/opt/data:/usr/local/apache2/htdocs
     |              |
     |              └── Container path — Apache document root
     └─────────────────── Host path — where static content is stored on stapp01
```

Files in `/opt/data` on the host are served directly by Apache as web content. No data was copied or modified — the volume mount creates a live link between the two paths.

#### `docker compose` Plugin vs `docker-compose` Binary

| Command          | Type                       | Installation                                                   |
| ---------------- | -------------------------- | -------------------------------------------------------------- |
| `docker-compose` | Standalone binary (legacy) | Separate install, Python-based                                 |
| `docker compose` | CLI plugin (modern)        | Bundled with Docker Engine via `docker-compose-plugin` package |

Docker officially deprecated the standalone `docker-compose` binary in favour of the `docker compose` plugin integrated directly into the Docker CLI. On CentOS/RHEL systems, the plugin is installed via `yum install docker-compose-plugin`.

#### Apache httpd Document Root

The `httpd:latest` image serves files from `/usr/local/apache2/htdocs` by default. Mounting `/opt/data` at this path means the pre-existing content in `/opt/data` on the host is served directly — the `index1.html` file visible in the `curl` response came from this mounted directory without any modification to the source data.

***

_Lab completed on 2026-03-26 | Server: stapp01 | OS: CentOS Stream 9 | Docker: 26.1.3 | Docker Compose: v5.1.1_

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
