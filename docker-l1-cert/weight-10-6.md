# Weight: 10

There is a static website running within a container named `nautilus`, this container is running on `App Server 2`. Suddenly, we started facing some issues with the static website on `App Server 2`. Look into the issue to fix the same, you can find more details below:

a. Container's volume `/usr/local/apache2/htdocs` is mapped with the host volume `/var/www/html`.\
b. The website should run on host port `8081` on `App Server 2` i.e command `curl http://localhost:8081/` should work on `App Server 2`.

***

### 9/9 – Fix Static Website on App Server 2

**Question:**\
There is a static website running within a container named `nautilus` on App Server 2. The website should run on host port **8081**. The container’s volume `/usr/local/apache2/htdocs` is mapped to host volume `/var/www/html`. The website was not accessible — fix the issue and ensure `curl http://localhost:8081/` works.

***

#### Steps and Terminal Output

**1. Check existing containers**

```bash
[steve@stapp02 ~]$ sudo docker ps -a
CONTAINER ID   IMAGE      COMMAND          STATUS                  PORTS   NAMES
5725d1fd48c3   httpd      "httpd-foreground" Exited (0) 32 seconds ago      nautilus
```

> The `nautilus` container is **exited** and has no port mapping.

***

**2. Remove the broken container**

```bash
[steve@stapp02 ~]$ sudo docker rm nautilus
nautilus
```

***

**3. Recreate the container with correct port and volume**

```bash
[steve@stapp02 ~]$ sudo docker run -d \
--name nautilus \
-p 8081:80 \
-v /var/www/html:/usr/local/apache2/htdocs \
httpd
3e3494680ffc
```

***

**4. Verify container is running with correct port**

```bash
[steve@stapp02 ~]$ sudo docker ps
CONTAINER ID   IMAGE      COMMAND          STATUS          PORTS                        NAMES
3e3494680ffc   httpd      "httpd-foreground" Up 6 seconds   0.0.0.0:8081->80/tcp        nautilus
```

***

**5. Test the website**

```bash
[steve@stapp02 ~]$ curl http://localhost:8081/
Welcome to KodeKloud!
```

***

✅ **Result:**

* Container **`nautilus`** is running.
* Host port **8081** mapped to container port 80.
* Volume `/var/www/html` mapped to `/usr/local/apache2/htdocs`.
* Static website is accessible on host.
