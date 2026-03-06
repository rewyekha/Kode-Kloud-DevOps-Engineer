# Day 41: Write a Docker File

As per recent requirements shared by the Nautilus application development team, they need custom images created for one of their projects. Several of the initial testing requirements have already been shared with the DevOps team.

Create a **Dockerfile** at:

```
/opt/docker/Dockerfile
```

on **App Server 1 in Stratos DC** with the following requirements:

1. Use **ubuntu:24.04** as the base image.
2. Install **apache2**.
3. Configure Apache to run on **port 8086**.
4. Do **not modify other Apache configuration settings** such as document root.

***

## Infrastructure Details

| Server     | Hostname                           | User | Purpose               |
| ---------- | ---------------------------------- | ---- | --------------------- |
| stapp01    | stapp01.stratos.xfusioncorp.com    | tony | Nautilus App Server 1 |
| jump\_host | jump\_host.stratos.xfusioncorp.com | thor | Jump Server           |

***

## Step 1 – Connect to Jump Host

```bash
ssh thor@jump_host.stratos.xfusioncorp.com
```

#### Terminal Output

```
thor@jump-host ~$ ssh thor@jump_host.stratos.xfusioncorp.com
thor@jump_host.stratos.xfusioncorp.com's password:
Last login: Fri Mar  6
thor@jump-host ~$
```

***

## Step 2 – Connect to App Server 1

```bash
ssh tony@stapp01
```

#### Terminal Output

```
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.240.128)' can't be established.
ED25519 key fingerprint is SHA256:MLprqeFoPiOWUDCyP//RuT1O5KEszBxDPITFWud8wys.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

***

## Step 3 – Create Docker Directory

```bash
sudo mkdir -p /opt/docker
```

#### Terminal Output

```
[tony@stapp01 ~]$ sudo mkdir -p /opt/docker

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

#1) Respect the privacy of others.
#2) Think before you type.
#3) With great power comes great responsibility.

[sudo] password for tony:
```

***

## Step 4 – Create Dockerfile

```bash
sudo vi /opt/docker/Dockerfile
```

Add the following content:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8086/g' /etc/apache2/ports.conf && \
    sed -i 's/:80/:8086/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8086

CMD ["apachectl", "-D", "FOREGROUND"]
```

Save the file using:

```
ESC
:wq
```

***

## Step 5 – Verify the Dockerfile

```bash
cat /opt/docker/Dockerfile
```

#### Terminal Output

```
[tony@stapp01 ~]$ cat /opt/docker/Dockerfile
FROM ubuntu:24.04

RUN apt-get update && \
    apt-get install -y apache2

RUN sed -i 's/Listen 80/Listen 8086/g' /etc/apache2/ports.conf && \
    sed -i 's/:80/:8086/g' /etc/apache2/sites-available/000-default.conf

EXPOSE 8086

CMD ["apachectl", "-D", "FOREGROUND"]
```

***

## Result

The **Dockerfile** has been successfully created at:

```
/opt/docker/Dockerfile
```

It satisfies all requirements:

* Base image: **ubuntu:24.04**
* **Apache2 installed**
* Apache configured to run on **port 8086**
* No other Apache configuration changes made.

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>
