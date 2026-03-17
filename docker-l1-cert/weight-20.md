# Weight: 20

The Nautilus DevOps team was testing a custom container on `Application Server 2` in `Stratos DC`. They were able to configure it as per their requirements, now they wanted to create an image from this container.

a. The name of the container is `alpine_nautilus`.

b. Create an image `alpine:nautilus` from this container.

```bash
[steve@stapp02 ~]$ sudo docker ps -a
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                               NAMES
04da0fa1b010   alpine                  "/bin/sleep 100000"      48 seconds ago   Up 47 seconds                                       alpine_nautilus
d8d47918300b   httpd:alpine            "httpd-foreground"       3 minutes ago    Up 3 minutes    80/tcp                              development_3
1878489387e3   ubuntu/apache2:latest   "apache2-foreground"     8 minutes ago    Up 8 minutes    80/tcp                              ubuntu_latest
076462effdb6   nginx:alpine            "/docker-entrypoint.…"   10 minutes ago   Up 10 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_2
5bb9a426f29c   ubuntu/apache2:latest   "sleep 1000"             13 minutes ago   Up 13 minutes   80/tcp                              debug_2
[steve@stapp02 ~]$ sudo docker commit alpine_nautilus alpine:nautilus
sha256:0c3615e827058494a9b21d5171b746613e436d6f87fd1d82d4e9287ef711db26
[steve@stapp02 ~]$ sudo docker images
REPOSITORY       TAG        IMAGE ID       CREATED         SIZE
alpine           nautilus   0c3615e82705   7 seconds ago   8.44MB
nginx            alpine     d0c780774910   34 hours ago    62.2MB
httpd            alpine     df4281218506   6 weeks ago     67.1MB
alpine           latest     a40c03cbb81c   6 weeks ago     8.44MB
ubuntu/apache2   latest     d9c1fa2be67f   3 months ago    156MB
[steve@stapp02 ~]$ 
```
