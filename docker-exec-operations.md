# Docker EXEC Operations

One of the Nautilus DevOps team members was working to configure services on a `kkloud` container that is running on `App Server 2` in `Stratos Datacenter`. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:

a. Install `apache2` in `kkloud` container using `apt` that is running on `App Server 2` in `Stratos Datacenter`.\
\
b. Configure Apache to listen on port `8085` instead of default `http` port. Do not bind it to listen on specific IP or hostname only, i.e it should listen on localhost, 127.0.0.1, container ip, etc.\
\
c. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.



***

## Docker: Execute Commands in Container and Configure Apache

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter\
> **Difficulty:** Beginner | **Topic:** Docker, apache2, docker exec, container configuration

***

### Table of Contents

1. [Lab Question](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-question)
2. [Infrastructure Details](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#infrastructure-details)
3. [Solution](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#solution)
   * [Step 1: SSH into Application Server 2](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-1-ssh-into-application-server-2)
   * [Step 2: Identify the Running Container](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-2-identify-the-running-container)
   * [Step 3: Access the Container Using docker exec](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-3-access-the-container-using-docker-exec)
   * [Step 4: Install Apache2](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-4-install-apache2)
   * [Step 5: Configure Apache to Use Port 8085](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-5-configure-apache-to-use-port-8085)
   * [Step 6: Start Apache Service](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-6-start-apache-service)
   * [Step 7: Verify Apache Service](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-7-verify-apache-service)
   * [Step 8: Ensure Container is Running](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#step-8-ensure-container-is-running)
4. [Lab Complete](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#lab-complete)
5. [Key Concepts](https://chatgpt.com/c/69c23ef8-1ae0-8323-80bb-0c3f7948a669#key-concepts)

***

### Lab Question

One of the Nautilus DevOps team members was working to configure services on a kkloud container that is running on App Server 2 in Stratos Datacenter. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:

Requirements:

1. Install apache2 in kkloud container using apt that is running on App Server 2 in Stratos Datacenter.
2. Configure Apache to listen on port 8085 instead of default http port. Do not bind it to listen on specific IP or hostname only.
3. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.

***

### Infrastructure Details

| Server Name          | Hostname  | User    | Password   | Purpose                      |
| -------------------- | --------- | ------- | ---------- | ---------------------------- |
| Application Server 1 | stapp01   | tony    | Ir0nM@n    | Hosts Nautilus Application 1 |
| Application Server 2 | stapp02   | steve   | Am3ric@    | Hosts Nautilus Application 2 |
| Application Server 3 | stapp03   | banner  | BigGr33n   | Hosts Nautilus Application 3 |
| LoadBalancer Server  | stlb01    | loki    | Mischi3f   | Distributes traffic          |
| Database Server      | stdb01    | peter   | Sp!dy      | Hosts database               |
| Storage Server       | ststor01  | natasha | Bl@kW      | Storage                      |
| Backup Server        | stbkp01   | clint   | H@wk3y3    | Backups                      |
| Mail Server          | stmail01  | groot   | Gr00T123   | Mail services                |
| Jump Host            | jump-host | thor    | mjolnir123 | Access server                |
| Jenkins Server       | jenkins   | jenkins | j@rv!s     | CI/CD                        |

Target: Application Server 2 (stapp02) — container kkloud

***

### Solution

#### Step 1: SSH into Application Server 2

Connect to the target server from the jump host to perform operations.

```bash
ssh steve@stapp02
```

Terminal Output:

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.234.216)' can't be established.
ED25519 key fingerprint is SHA256:qQrOCBtpiC11vHutpYM9PEvSx14NgXDSl7ysmXNp2hQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
[steve@stapp02 ~]$
```

This confirms successful login to the target server.

***

#### Step 2: Identify the Running Container

Check for running containers to locate kkloud.

```bash
docker ps
```

Terminal Output:

```bash
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS     NAMES
574b1eaa7246   ubuntu:18.04   "/bin/bash"   2 minutes ago   Up 2 minutes             kkloud
```

The container kkloud is running and ready for configuration.

***

#### Step 3: Access the Container Using docker exec

Enter the container to perform installation and configuration.

```bash
docker exec -it 574b1eaa7246 bash
```

Terminal Output:

```bash
root@574b1eaa7246:/#
```

The shell prompt confirms access inside the container.

***

#### Step 4: Install Apache2

Install apache2 using apt package manager.

```bash
apt update
apt install apache2
```

Terminal Output:

```bash
...
Setting up apache2 (2.4.29-1ubuntu4.27) ...
invoke-rc.d: policy-rc.d denied execution of start.
...
```

Apache is installed successfully. Service did not auto-start due to container policy.

***

#### Step 5: Configure Apache to Use Port 8085

Modify Apache configuration to change default port.

```bash
sed -i 's/Listen 80/Listen 8085/g' /etc/apache2/ports.conf
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8085>/g' /etc/apache2/sites-available/000-default.conf
```

Terminal Output:

```bash
root@574b1eaa7246:/#
```

Configuration updated successfully.

***

#### Step 6: Start Apache Service

Start Apache manually since system services are restricted in containers.

```bash
apachectl start
```

Terminal Output:

```bash
AH00558: apache2: Could not reliably determine the server's fully qualified domain name
```

Apache starts successfully. Warning is non-critical.

***

#### Step 7: Verify Apache Service

Confirm Apache is working by accessing the service.

```bash
curl http://localhost:8085
```

Terminal Output:

```bash
Apache2 Ubuntu Default Page
...
It works!
```

The default page confirms Apache is serving on port 8085.

***

#### Step 8: Ensure Container is Running

Exit container and confirm it remains active.

```bash
exit
docker ps
```

Terminal Output:

```bash
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS     NAMES
574b1eaa7246   ubuntu:18.04   "/bin/bash"   7 minutes ago   Up 7 minutes             kkloud
```

Container remains in running state.

***

### Lab Complete

| Task                | Detail                         | Status    |
| ------------------- | ------------------------------ | --------- |
| Apache installation | Installed via apt              | Completed |
| Port configuration  | Changed to 8085                | Completed |
| Apache service      | Running via apachectl          | Completed |
| Service validation  | curl localhost:8085 successful | Completed |
| Container state     | kkloud running                 | Completed |

***

### Key Concepts

#### docker exec

`docker exec` allows execution of commands inside a running container.

```bash
docker exec -it <container_id> bash
```

It provides interactive access without stopping or restarting the container.

***

#### Apache Port Configuration

Apache listens on ports defined in:

* `/etc/apache2/ports.conf`
* `/etc/apache2/sites-available/000-default.conf`

Both must be updated to avoid mismatch issues.

***

#### Service Management in Containers

Containers typically do not run systemd or init systems.

| Command   | Works in Container |
| --------- | ------------------ |
| systemctl | No                 |
| service   | Sometimes          |
| apachectl | Yes                |

`apachectl` directly controls Apache without relying on system services.

***

#### policy-rc.d Restriction

The message:

```bash
policy-rc.d denied execution of start
```

Prevents automatic service start during package installation. This is common in containers to avoid unintended daemon execution.

***

#### Verification Using curl

`curl` is used to test HTTP service availability.

```bash
curl http://localhost:8085
```

Successful response confirms:

* Service is running
* Port configuration is correct
* Networking is functional

***

_Lab completed on 2026-03-24 | Server: stapp02 | OS: Ubuntu 18.04 (Container) | Docker Container_

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
