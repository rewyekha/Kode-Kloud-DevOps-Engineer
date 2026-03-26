# Complete Overview

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Docker Lab Overview – Nautilus Tasks 1–9

This guide covers practical Docker skills demonstrated on **Application Server 2** in Stratos Datacenter. Each task highlights key concepts, commands, and best practices.

***

#### **Task 1 – Run a Debug Container with Custom CMD**

**Objective:**\
Create a container named `debug_2` using image `ubuntu/apache2:latest` and override the default CMD.

**Key Concepts:**

* `docker run` with `--name`
* Overriding default CMD
* Running container in **detached mode** (`-d`)

**Commands & Output:**

```bash
sudo docker pull ubuntu/apache2:latest
sudo docker run -d --name debug_2 ubuntu/apache2:latest sleep 1000
sudo docker ps
```

**Result:**\
Container `debug_2` is running with `sleep 1000` as the CMD.

***

#### **Task 2 – Deploy nginx Container**

**Objective:**\
Deploy `nginx_2` using `nginx:alpine` image and ensure it’s running.

**Key Concepts:**

* Deploying lightweight nginx container
* Detached mode
* Port mapping optional if default HTTP testing not required

**Commands:**

```bash
sudo docker run -d --name nginx_2 nginx:alpine
sudo docker ps
```

***

#### **Task 3 – Copy File from Host to Container**

**Objective:**\
Copy `/tmp/nautilus.txt.gpg` from Docker host to `ubuntu_latest` container in `/home/`.

**Key Concepts:**

* `docker cp` for copying files **host ↔ container**
* Creating directories inside container if not exist

**Commands:**

```bash
sudo docker exec ubuntu_latest mkdir -p /home
sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/home/
sudo docker exec ubuntu_latest ls -l /home/
```

**Result:**\
File copied safely without modification.

***

#### **Task 4 – Copy File from Container to Host**

**Objective:**\
Copy `/tmp/test.txt.gpg` from container `development_3` to Docker host `/tmp`.

**Key Concepts:**

* Reverse use of `docker cp`
* Check container status with `docker ps -a` before copying

**Commands:**

```bash
sudo docker cp development_3:/tmp/test.txt.gpg /tmp/
ls -l /tmp/test.txt.gpg
```

***

#### **Task 5 – Create Image from Container**

**Objective:**\
Create a custom image `alpine:nautilus` from container `alpine_nautilus`.

**Key Concepts:**

* `docker commit` creates an image from container state
* Verify new image with `docker images`

**Commands:**

```bash
sudo docker commit alpine_nautilus alpine:nautilus
sudo docker images
```

***

#### **Task 6 – Cleanup Unused Images**

**Objective:**\
Remove images `alpine:3.18.4` and `sebp/lighttpd:latest`. Remove containers using them first.

**Key Concepts:**

* Check containers using an image: `docker ps -a`
* Remove container: `docker rm <container>`
* Remove image: `docker rmi <image>`

**Commands:**

```bash
sudo docker rm lighttpd
sudo docker rmi sebp/lighttpd:latest
sudo docker rmi alpine:3.18.4
sudo docker images
```

***

#### **Task 7 – Create Custom Docker Network**

**Objective:**\
Create `mysql-network` with bridge driver, subnet `182.18.0.0/24`, gateway `182.18.0.1`.

**Key Concepts:**

* `docker network create` with custom subnet and gateway
* Useful for multi-container setups

**Commands:**

```bash
sudo docker network create \
--driver bridge \
--subnet 182.18.0.0/24 \
--gateway 182.18.0.1 \
mysql-network
sudo docker network inspect mysql-network
```

***

#### **Task 8 – Remove Unused Docker Network**

**Objective:**\
Delete network `php-network`.

**Key Concepts:**

* Removing unused networks: `docker network rm`
* Verify with `docker network ls`

**Commands:**

```bash
sudo docker network rm php-network
sudo docker network ls
```

***

#### **Task 9 – Fix Static Website Container**

**Objective:**\
Fix container `nautilus` so website works on host port **8081**, volume mapping `/var/www/html:/usr/local/apache2/htdocs`.

**Key Concepts:**

* Port mapping: `-p <host_port>:<container_port>`
* Volume mapping: `-v <host_dir>:<container_dir>`
* Remove broken container and recreate with correct settings

**Commands:**

```bash
sudo docker rm nautilus
sudo docker run -d \
--name nautilus \
-p 8081:80 \
-v /var/www/html:/usr/local/apache2/htdocs \
httpd
curl http://localhost:8081/
```

**Output:**

```
Welcome to KodeKloud!
```

***

### **Summary – Key Docker Concepts Covered**

| Concept             | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| Container Lifecycle | `docker run`, `docker stop`, `docker rm`, `docker ps`                  |
| Images              | `docker pull`, `docker commit`, `docker rmi`, `docker images`          |
| Volume Mapping      | `-v host_dir:container_dir` to persist data                            |
| Port Mapping        | `-p host_port:container_port` for network access                       |
| File Copy           | `docker cp` for host ↔ container                                       |
| Networks            | `docker network create`, `docker network inspect`, `docker network rm` |
| Detached Mode       | `-d` to run containers in background                                   |
| Troubleshooting     | Check exited containers, recreate with proper settings                 |

***

✅ This overview captures **all 9 tasks**, terminal commands, and outcomes. It can be used as a **reference or study guide** for Docker practical labs.
