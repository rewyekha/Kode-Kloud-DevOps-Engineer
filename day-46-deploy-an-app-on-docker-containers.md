# Day 46: Deploy an App on Docker Containers

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

## **Lab: Deploy Nautilus App Using Docker Compose**

### **Lab Objective**

The Nautilus Application team wants to deploy a containerized stack using Docker Compose. You will:

* Create a `docker-compose.yml` file on **stapp02**
* Deploy a web (`php_blog`) and database (`mysql_blog`) service
* Map volumes and ports
* Verify the app and database are running

***

### **Question / Task**

**Task:**

On App Server 2 (`stapp02`), create a Docker Compose stack with:

#### **Web Service**

* Container name: `php_blog`
* Image: `php:apache`
* Host port 3000 → container port 80
* Volume: `/var/www/html` → container `/var/www/html`

#### **DB Service**

* Container name: `mysql_blog`
* Image: `mariadb:latest`
* Host port 3306 → container port 3306
* Volume: `/var/lib/mysql` → container `/var/lib/mysql`
* Database: `database_blog`
* Custom user: `bloguser` with a complex password

**Verify:**\
Access the web app using:

```bash
curl http://<server-ip>:3000/
```

***

### **Step 1: SSH into stapp02**

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.244.135)' can't be established.
ED25519 key fingerprint is SHA256:hZWWAbyQkkCDiUpNX3XKrJSRnWnB6I4t4CyCLo6avqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
```

***

### **Step 2: Create directories for volumes**

```bash
[steve@stapp02 ~]$ sudo mkdir -p /var/www/html
[steve@stapp02 ~]$ sudo mkdir -p /var/lib/mysql
```

Set ownership:

```bash
[steve@stapp02 ~]$ sudo chown -R $(whoami):$(whoami) /var/www/html /var/lib/mysql
```

***

### **Step 3: Create Docker Compose directory and file**

```bash
[steve@stapp02 ~]$ sudo mkdir -p /opt/itadmin
[steve@stapp02 ~]$ sudo nano /opt/itadmin/docker-compose.yml
```

Paste the following content:

```yaml
version: "3.8"

services:
  web:
    container_name: php_blog
    image: php:apache
    ports:
      - "3000:80"
    volumes:
      - /var/www/html:/var/www/html
    restart: unless-stopped

  db:
    container_name: mysql_blog
    image: mariadb:latest
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: database_blog
      MYSQL_USER: bloguser
      MYSQL_PASSWORD: Str0ngP@ssw0rd!
      MYSQL_ROOT_PASSWORD: RootP@ss123
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    restart: unless-stopped
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

***

### **Step 4: Deploy the stack using Docker Compose plugin**

```bash
[steve@stapp02 itadmin]$ docker compose up -d
WARN[0000] /opt/itadmin/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] up 27/27
 ✔ Image mariadb:latest    Pulled                                             12.3s
 ✔ Image php:apache        Pulled                                             14.9s
 ✔ Network itadmin_default Created                                            0.1s
 ✔ Container php_blog      Created                                            0.2s
 ✔ Container mysql_blog    Created                                            0.3s
```

***

### **Step 5: Verify containers are running**

```bash
[steve@stapp02 itadmin]$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                       NAMES
cf803e10c0de   mariadb:latest   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp   mysql_blog
f0fdbfd4cf84   php:apache       "docker-php-entrypoi…"   8 seconds ago   Up 7 seconds   0.0.0.0:3000->80/tcp, :::3000->80/tcp       php_blog
```

***

### **Step 6: Test the web app**

From the server:

```bash
[steve@stapp02 itadmin]$ curl http://localhost:3000/
<html>
    <head>
        <title>Welcome to xFusionCorp Industries!</title>
    </head>
    <body>
        Welcome to xFusionCorp Industries!
    </body>
</html>
```

From another host (using server IP):

```bash
[steve@stapp02 itadmin]$ curl http://10.244.244.135:3000/
<html>
    <head>
        <title>Welcome to xFusionCorp Industries!</title>
    </head>
    <body>
        Welcome to xFusionCorp Industries!
    </body>
</html>
```

***

### **Step 7: Connect to MariaDB database**

```bash
[steve@stapp02 itadmin]$ docker exec -it mysql_blog mariadb -u bloguser -p
Enter password: Str0ngP@ssw0rd!
Welcome to the MariaDB monitor.  Commands end with ; or \g.
MariaDB [(none)]> SHOW DATABASES;
+------------------+
| Database         |
+------------------+
| database_blog    |
| information_schema |
+------------------+
```

Switch to the database:

```sql
MariaDB [(none)]> USE database_blog;
MariaDB [database_blog]> CREATE TABLE test_table (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50)
);
MariaDB [database_blog]> INSERT INTO test_table (name) VALUES ('LabTest');
MariaDB [database_blog]> SELECT * FROM test_table;
+----+---------+
| id | name    |
+----+---------+
| 1  | LabTest |
+----+---------+
```

Exit:

```sql
MariaDB [database_blog]> exit
```

***

### ** Lab Verification Checklist**

| Requirement | Status |
| --------------------------------------------------- | ------ |
| `/opt/itadmin/docker-compose.yml` exists | |
| Two services deployed (`php_blog` and `mysql_blog`) | |
| Correct port mapping (3000→80, 3306→3306) | |
| Volume mapping for persistence | |
| Database `database_blog` created | |
| Custom user `bloguser` can connect | |
| Web app accessible via `curl` | |

***

### **Step 8: Cleanup (Optional)**

If needed to stop and remove the stack:

```bash
docker compose down
```

***

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
