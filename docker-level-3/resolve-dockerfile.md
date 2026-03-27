# Resolve Dockerfile

### Debugging a Dockerfile for Apache

#### **Objective**

* Fix a failing Dockerfile located at `/opt/docker`.
* Build an image without changing base image, content, or data files.

***

#### **Original Dockerfile Issues**

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf.d/httpd.conf
RUN sed -i '/LoadModule\ ssl_module modules\/mod_ssl.so/s/^#//g' conf.d/httpd.conf
RUN sed -i '/LoadModule\ socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' conf.d/httpd.conf
RUN sed -i '/Include\ conf\/extra\/httpd-ssl.conf/s/^#//g' conf.d/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key
COPY html/index.html /usr/local/apache2/htdocs/
```

**Problem:**

* The Apache image uses `/usr/local/apache2/conf/httpd.conf`.
* `conf.d/httpd.conf` **does not exist**, causing the build to fail.

***

#### **Fixed Dockerfile**

```dockerfile
FROM httpd:2.4.43

RUN sed -i "s/Listen 80/Listen 8080/g" /usr/local/apache2/conf/httpd.conf
RUN sed -i '/LoadModule ssl_module modules\/mod_ssl.so/s/^#//g' /usr/local/apache2/conf/httpd.conf
RUN sed -i '/LoadModule socache_shmcb_module modules\/mod_socache_shmcb.so/s/^#//g' /usr/local/apache2/conf/httpd.conf
RUN sed -i '/Include conf\/extra\/httpd-ssl.conf/s/^#//g' /usr/local/apache2/conf/httpd.conf

COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key
COPY html/index.html /usr/local/apache2/htdocs/
```

***

#### **Building the Image**

```bash
docker build -t httpd-fixed-kke .
docker images
```

**Expected Output:**

```
REPOSITORY        TAG       IMAGE ID       CREATED          SIZE
httpd-fixed-kke   latest    870ddd28f411   25 seconds ago   166MB
```

***

#### **Key Notes**

1. Paths must match the base image (`/usr/local/apache2/conf/httpd.conf`).
2. Image name/tag is irrelevant for the lab – the system rebuilds automatically on “Finish”.
3. Verify file existence (`certs` and `html`) before building.
4. Dockerfile build success is the **main success criteria**.

***

### 🔑 Pro Tips / Takeaways

* **`docker run` vs `docker create`**
  * Use `run` to create + start container in one step.
  * Use `create` only if you plan to start later.
* **Volume mapping**
  * Host → container mapping must be absolute paths.
  * Files copied to host appear inside the container automatically.
* **Image transfer**
  * `docker save` → `scp` → `docker load`.
  * Verify with `ls` and `docker images`.
* **Dockerfile debugging**
  * Check file paths inside image.
  * Use small `RUN` tests before `COPY`.
  * Build success = exam success.
*   **Golden rule for labs**

    | Question Type                   | What Matters             |
    | ------------------------------- | ------------------------ |
    | Fix Dockerfile                  | Build success            |
    | Create container / port mapping | Names and ports matter   |
    | Transfer image                  | File exists + load works |

***

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>
