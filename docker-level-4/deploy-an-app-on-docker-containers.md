# Deploy an App on Docker Containers

## Docker Compose: Deploy PHP Apache and MariaDB Stack

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker Compose, PHP, Apache, MariaDB, Multi-service Stack

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 1](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-1)
   * [Step 2: Verify the Directory Structure](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-verify-the-directory-structure)
   * [Step 3: Navigate to the Security Directory](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-navigate-to-the-security-directory)
   * [Step 4: Create the Docker Compose File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-create-the-docker-compose-file)
   * [Step 5: Verify the Compose File](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-verify-the-compose-file)
   * [Step 6: Deploy the Stack](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-deploy-the-stack)
   * [Step 7: Test the Web Service](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-test-the-web-service)
4. [Final docker-compose.yml](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#final-docker-composeyml)
5. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus Application development team recently finished development of an app they want to deploy on a containerized platform. The team wants to test the deployment on App Server 1 before going live by setting up a complete containerized stack using a Docker Compose file.

**Requirements:**

1. On **App Server 1** in Stratos Datacenter create a docker compose file `/opt/security/docker-compose.yml` (must be named exactly).
2. The compose should deploy two services (`web` and `db`), each deploying a container per the details below:

**For web service:**

* Container name: `php_host`
* Image: `php` with any `apache` tag
* Map container port `80` to host port `8087`
* Map container volume `/var/www/html` to host volume `/var/www/html`

**For DB service:**

* Container name: `mysql_host`
* Image: `mariadb` with any tag (preferably `latest`)
* Map container port `3306` to host port `3306`
* Map container volume `/var/lib/mysql` to host volume `/var/lib/mysql`
* Set `MYSQL_DATABASE=database_host` and use a custom user (not root) with a complex password

3. After running `docker compose up`, the app should be accessible via `curl <server-ip>:8087/`

> **Note:** Once you click FINISH, all currently running/stopped containers will be destroyed and the stack will be deployed again using your compose file.

***

> **Target:** Application Server 1 (`stapp01`) — compose file must be created at `/opt/security/docker-compose.yml`.

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
The authenticity of host 'stapp01 (10.244.13.29)' can't be established.
ED25519 key fingerprint is SHA256:NxZoojwxrXtY3wdICklpmrLBsy1YIBRFsbM6nTUb04Y.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 2: Verify the Directory Structure

Check whether the `/opt/security` directory already exists:

```bash
ls /opt/security/
ls /opt/
```

**Terminal Output:**

```
[tony@stapp01 ~]$ ls /opt/security/
[tony@stapp01 ~]$ ls /opt/
containerd  security
```

The `/opt/security/` directory already exists and is empty. An attempt was made to create it with `sudo mkdir -p /opt/security` but this was unnecessary — the directory was already present. The first `sudo` attempt also encountered a password entry issue but succeeded on retry:

```
[tony@stapp01 ~]$ sudo mkdir -p /opt/security
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
Sorry, try again.
[sudo] password for tony:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 3: Navigate to the Security Directory

```bash
cd /opt/security/
ls
```

**Terminal Output:**

```bash
[tony@stapp01 ~]$ cd /opt/security/
[tony@stapp01 security]$ ls
[tony@stapp01 security]$
```

The directory is empty and ready for the compose file.

***

#### Step 4: Create the Docker Compose File

Create the compose file using `sudo vi`:

```bash
sudo vi docker-compose.yml
```

**Terminal Output:**

```
[tony@stapp01 security]$ sudo vi docker-compose.yml
[sudo] password for tony:
[tony@stapp01 security]$
```

Enter the following content in the editor:

```yaml
services:
  web:
    image: php:8.2-apache
    container_name: php_host
    ports:
      - "8087:80"
    volumes:
      - /var/www/html:/var/www/html
  db:
    image: mariadb:latest
    container_name: mysql_host
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: dbuser
      MYSQL_PASSWORD: StrongPass@123
      MYSQL_ROOT_PASSWORD: RootPass@123
```

Save and exit: `Esc` → `:wq` → `Enter`

***

#### Step 5: Verify the Compose File

Confirm the file contents are correct before deploying:

```bash
cat docker-compose.yml
```

**Terminal Output:**

```bash
[tony@stapp01 security]$ cat docker-compose.yml
services:
  web:
    image: php:8.2-apache
    container_name: php_host
    ports:
      - "8087:80"
    volumes:
      - /var/www/html:/var/www/html
  db:
    image: mariadb:latest
    container_name: mysql_host
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: dbuser
      MYSQL_PASSWORD: StrongPass@123
      MYSQL_ROOT_PASSWORD: RootPass@123
```

All configuration values match the lab requirements:

| Requirement        | Configured Value                |
| ------------------ | ------------------------------- |
| Web container name | `php_host`                      |
| Web image          | `php:8.2-apache`                |
| Web port mapping   | `8087:80`                       |
| Web volume         | `/var/www/html:/var/www/html`   |
| DB container name  | `mysql_host`                    |
| DB image           | `mariadb:latest`                |
| DB port mapping    | `3306:3306`                     |
| DB volume          | `/var/lib/mysql:/var/lib/mysql` |
| MYSQL\_DATABASE    | `database_host`                 |
| MYSQL\_USER        | `dbuser` (non-root)             |

***

#### Step 6: Deploy the Stack

Start both containers in detached mode:

```bash
sudo docker compose up -d
```

**Terminal Output:**

```bash
[tony@stapp01 security]$ sudo docker compose up -d
[+] up 28/28
 ✔ Image php:8.2-apache     Pulled                                                                                    13.3s
 ✔ Image mariadb:latest     Pulled                                                                                    10.9s
 ✔ Network security_default Created                                                                                   0.1s
 ✔ Container php_host       Created                                                                                   0.2s
 ✔ Container mysql_host     Created                                                                                   0.2s
```

Docker Compose performed five actions:

1. `php:8.2-apache` image pulled in `13.3s`
2. `mariadb:latest` image pulled in `10.9s`
3. Default network `security_default` created in `0.1s`
4. Container `php_host` created in `0.2s`
5. Container `mysql_host` created in `0.2s`

All 28 sub-tasks completed successfully — `[+] up 28/28`.

***

#### Step 7: Test the Web Service

Send a `curl` request to port `8087` to confirm the PHP Apache web server is serving content:

```bash
curl http://localhost:8087/
```

**Terminal Output:**

```bash
[tony@stapp01 security]$ curl http://localhost:8087/
<html>
    <head>
        <title>Welcome to xFusionCorp Industries!</title>
    </head>
    <body>
        Welcome to xFusionCorp Industries!    </body>
</html>
[tony@stapp01 security]$
```

The web server responded with the xFusionCorp Industries HTML page — confirming:

* The `php_host` container is running and serving HTTP traffic on port `8087`
* The volume mount `/var/www/html` is serving the pre-existing HTML content correctly
* The port mapping `8087:80` is forwarding host traffic to the Apache container

***

### Final docker-compose.yml

```yaml
services:
  web:
    image: php:8.2-apache
    container_name: php_host
    ports:
      - "8087:80"
    volumes:
      - /var/www/html:/var/www/html

  db:
    image: mariadb:latest
    container_name: mysql_host
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: dbuser
      MYSQL_PASSWORD: StrongPass@123
      MYSQL_ROOT_PASSWORD: RootPass@123
```

***

### Lab Complete

| Requirement        | Detail                               | Status    |
| ------------------ | ------------------------------------ | --------- |
| Compose file path  | `/opt/security/docker-compose.yml`   | Confirmed |
| Web container name | `php_host`                           | Confirmed |
| Web image          | `php:8.2-apache`                     | Confirmed |
| Web port           | `8087` → `80`                        | Confirmed |
| Web volume         | `/var/www/html:/var/www/html`        | Confirmed |
| DB container name  | `mysql_host`                         | Confirmed |
| DB image           | `mariadb:latest`                     | Confirmed |
| DB port            | `3306` → `3306`                      | Confirmed |
| DB volume          | `/var/lib/mysql:/var/lib/mysql`      | Confirmed |
| `MYSQL_DATABASE`   | `database_host`                      | Confirmed |
| `MYSQL_USER`       | `dbuser` (non-root)                  | Confirmed |
| Web response       | `Welcome to xFusionCorp Industries!` | Confirmed |

***

### Key Concepts

#### Two-Service Stack Architecture

This compose file defines a classic web + database two-tier architecture:

```bash
Host (stapp01)
├── Port 8087 ──▶ php_host (php:8.2-apache) ──▶ Port 80
│                   Volume: /var/www/html
│
└── Port 3306 ──▶ mysql_host (mariadb:latest) ──▶ Port 3306
                    Volume: /var/lib/mysql
                    Env: MYSQL_DATABASE, MYSQL_USER, MYSQL_PASSWORD
```

The two containers share the automatically created `security_default` network, allowing the `php_host` container to reach `mysql_host` by its service name `db` or container name `mysql_host` as a DNS hostname within the Docker network.

#### Why Four MariaDB Environment Variables

MariaDB requires four environment variables to initialise correctly:

| Variable              | Value            | Purpose                                      |
| --------------------- | ---------------- | -------------------------------------------- |
| `MYSQL_DATABASE`      | `database_host`  | Creates this database on first start         |
| `MYSQL_USER`          | `dbuser`         | Creates this non-root user                   |
| `MYSQL_PASSWORD`      | `StrongPass@123` | Password for `dbuser`                        |
| `MYSQL_ROOT_PASSWORD` | `RootPass@123`   | Required — MariaDB will not start without it |

`MYSQL_ROOT_PASSWORD` is mandatory even if you plan to use only the custom user. Without it, the MariaDB container exits immediately.

#### Host Volume Mounts for Persistence

Both services use **bind mounts** that map host directories directly into the containers:

```yaml
volumes:
  - /var/www/html:/var/www/html    # web content persists on host
  - /var/lib/mysql:/var/lib/mysql  # database files persist on host
```

This means that if the containers are stopped and recreated (as will happen when the FINISH button is clicked), the data and web content are preserved on the host filesystem and immediately available to the new containers. This is preferable to `emptyDir`-style anonymous volumes for stateful services.

#### PHP Apache Image Tag Selection

The `php` image offers multiple Apache-based tags:

| Tag              | PHP Version            | Use Case                      |
| ---------------- | ---------------------- | ----------------------------- |
| `php:8.2-apache` | 8.2 (used in this lab) | Modern stable PHP with Apache |
| `php:8.3-apache` | 8.3                    | Latest stable PHP             |
| `php:7.4-apache` | 7.4                    | Legacy applications           |
| `php:apache`     | Latest PHP             | Always pulls latest           |

`php:8.2-apache` was selected as a specific, stable tag. Using `php:apache` without a version would pull whatever latest is at deploy time — which could cause unexpected behaviour if the PHP version changes.

#### `security_default` Network

When Docker Compose starts, it automatically creates a network named after the directory containing the compose file — in this case `security_default` (from `/opt/security/`). Both containers are attached to this network by default, enabling:

* Container-to-container DNS resolution using service names (`web`, `db`)
* Network isolation from other Docker containers on the host
* No manual network configuration required

***

_Lab completed on 2026-03-28 | Server: stapp01 | OS: CentOS Stream 9 | Docker Compose: v5.0.2_

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
