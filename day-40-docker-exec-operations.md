# Day 40: Docker EXEC Operations

One of the Nautilus DevOps team members was working to configure services on a `kkloud` container that is running on `App Server 2` in `Stratos Datacenter`. Due to some personal work he is on PTO for the rest of the week, but we need to finish his pending work ASAP. Please complete the remaining work as per details given below:

a. Install `apache2` in `kkloud` container using `apt` that is running on `App Server 2` in `Stratos Datacenter`.

b. Configure Apache to listen on port `6200` instead of default `http` port. Do not bind it to listen on specific IP or hostname only, i.e it should listen on localhost, 127.0.0.1, container ip, etc.

c. Make sure Apache service is up and running inside the container. Keep the container in running state at the end.



***

## Deploy Apache on Custom Port Inside Docker Container

### Question

Connect to the application server and configure an Apache web server inside the running Docker container.

#### Requirements

* SSH into the application server **stapp02**.
* Access the running Docker container named **kkloud**.
* Install **Apache2** inside the container.
* Configure Apache to run on **port 6200 instead of port 80**.
* Start the Apache service.
* Verify the service is working by accessing the web page.

***

## Step 1: Connect to Application Server

SSH from the jump host to the application server.

```bash
thor@jumphost ~$ ssh steve@stapp02
```

Output:

```bash
The authenticity of host 'stapp02 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:oYVV6UcxqgcM8LUMzEXWlBW8P+9nC9N2niIsIs1PsB0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
```

***

## Step 2: Verify Running Docker Container

Check if the container is running.

```bash
docker ps
```

Output:

```bash
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS         PORTS     NAMES
470187c1c6ac   ubuntu:18.04   "/bin/bash"   2 minutes ago   Up 2 minutes             kkloud
```

***

## Step 3: Access the Container

Connect to the container using `docker exec`.

```bash
docker exec -it kkloud bash
```

Output:

```bash
root@470187c1c6ac:/#
```

***

## Step 4: Update Package Repository

```bash
apt update
```

Output (shortened):

```bash
Hit:1 http://archive.ubuntu.com/ubuntu bionic InRelease
Hit:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
Reading package lists... Done
```

***

## Step 5: Install Apache

```bash
apt install -y apache2
```

Output (shortened):

```bash
The following NEW packages will be installed:
apache2 apache2-bin apache2-data apache2-utils
...
Setting up apache2 (2.4.29-1ubuntu4.27) ...
```

***

## Step 6: Modify Apache Port

The default Apache port is **80**.\
We need to change it to **6200**.

#### Update ports.conf

```bash
sed -i 's/Listen 80/Listen 6200/g' /etc/apache2/ports.conf
```

#### Update Virtual Host Configuration

```bash
sed -i 's/<VirtualHost \*:80>/<VirtualHost *:6200>/g' /etc/apache2/sites-available/000-default.conf
```

***

## Step 7: Start Apache Service

```bash
service apache2 start
```

Output:

```bash
Starting Apache httpd web server apache2
AH00558: apache2: Could not reliably determine the server's fully qualified domain name
```

This warning is normal and does not affect the service.

***

## Step 8: Verify Apache Service

Attempt to check the port using `netstat`.

```bash
netstat -tulpn | grep 6200
```

Output:

```bash
bash: netstat: command not found
```

Attempt with `ss`.

```bash
ss -tulpn | grep 6200
```

Output:

```bash
bash: ss: command not found
```

Minimal containers often do not include these tools.

***

## Step 9: Verify Using Curl

Test the web server directly.

```bash
curl localhost:6200
```

Output (shortened):

```html
<title>Apache2 Ubuntu Default Page: It works</title>

Apache2 Ubuntu Default Page

If you can read this page, it means that the Apache HTTP server installed at
this site is working properly.
```

This confirms Apache is running on **port 6200**.

***

## Step 10: Check Service Status

```bash
service apache2 status
```

Output:

```bash
* apache2 is running
```

***

## Tools Used in This Lab

| Tool        | Purpose                                |
| ----------- | -------------------------------------- |
| SSH         | Remote login to the application server |
| Docker      | Manage containers                      |
| docker exec | Access running container               |
| apt         | Install packages                       |
| Apache2     | Web server                             |
| sed         | Modify configuration files             |
| curl        | Test HTTP service                      |
| service     | Manage system services                 |

***

## Tools That May Be Used in Future Labs

These tools are commonly used in troubleshooting and DevOps environments.

| Tool           | Purpose                            |
| -------------- | ---------------------------------- |
| netstat        | Display network connections        |
| ss             | Modern replacement for netstat     |
| vim / vi       | Edit configuration files           |
| nano           | Simple text editor                 |
| systemctl      | Manage services on systemd systems |
| ps             | View running processes             |
| grep           | Filter command output              |
| docker logs    | View container logs                |
| docker inspect | Inspect container configuration    |

***

## Key Takeaways

* Apache default port is **80**, but it can be changed in:
  * `/etc/apache2/ports.conf`
  * `/etc/apache2/sites-available/000-default.conf`
* Minimal Docker images may not contain troubleshooting tools like:
  * `netstat`
  * `ss`
* **curl** is a reliable way to test web servers quickly.
* `sed` is very useful for **automating configuration changes**.
