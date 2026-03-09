# Day 44: Write a Docker Compose File

## Hosting Static Website on Dockerized Apache HTTP Server

### Objective

Deploy a **static website** provided by the Nautilus application team using the `httpd` container on **App Server 2 (stapp02)**. The container should use Docker Compose with specific port and volume mappings.

***

### Environment Details

| Server Name | IP      | Hostname                        | User  | Purpose        |
| ----------- | ------- | ------------------------------- | ----- | -------------- |
| stapp02     | Dynamic | stapp02.stratos.xfusioncorp.com | steve | Nautilus App 2 |

**Docker host directory**: `/opt/docker`\
**Website content directory**: `/opt/dba`

***

### Requirements

1. On **App Server 2**, create a container named `httpd` using `/opt/docker/docker-compose.yml`.
2. Use the image `httpd:latest`.
3. Map container port `80` → host port `3000`.
4. Map container volume `/usr/local/apache2/htdocs` → host `/opt/dba`.
5. Do **not modify any data** within these locations.

***

### Steps and Terminal Outputs

#### 1. SSH to App Server 2

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.81.6)' can't be established.
ED25519 key fingerprint is SHA256:v5+hS5sJD2ExO5M7+EkEn9ktl83+a9xBi3amrhDcR1g.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
```

***

#### 2. Navigate to Docker directory

```bash
[steve@stapp02 ~]$ cd /opt/docker
[steve@stapp02 docker]$ ls
```

_(Directory was empty initially)_

***

#### 3. Create `docker-compose.yml`

```bash
[steve@stapp02 docker]$ sudo vi /opt/docker/docker-compose.yml
```

**File content:**

```yaml
version: '3'

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "3000:80"
    volumes:
      - /opt/dba:/usr/local/apache2/htdocs
```

***

#### 4. Start the container

```bash
[steve@stapp02 docker]$ sudo docker compose up -d
WARN[0000] /opt/docker/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 9/9
 ✔ Image httpd:latest     Pulled                                               3.7s
 ✔ Network docker_default Created                                              0.1s
 ✔ Container httpd        Created                                              0.1s
```

***

#### 5. Verify running container

```bash
[steve@stapp02 docker]$ docker ps
CONTAINER ID   IMAGE          COMMAND              CREATED          STATUS          PORTS                                   NAMES
d57d1897f9ec   httpd:latest   "httpd-foreground"   15 seconds ago   Up 14 seconds   0.0.0.0:3000->80/tcp, :::3000->80/tcp   httpd
```

***

#### 6. Verify volume mapping

```bash
[steve@stapp02 docker]$ docker inspect httpd | grep dba
                "/opt/dba:/usr/local/apache2/htdocs:rw"
                "Source": "/opt/dba",
```

***

#### 7. Test HTTP access

```bash
[steve@stapp02 docker]$ curl http://localhost:3000
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
```

**Result**: The Apache container is serving the static website content from `/opt/dba` successfully.

***

### ✅ Summary

* Docker container `httpd` created with `httpd:latest`.
* Container port `80` mapped to host port `3000`.
* Volume `/opt/dba` mapped correctly to `/usr/local/apache2/htdocs`.
* Website content is accessible via `curl` or browser at `http://stapp02:3000`.

***

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

