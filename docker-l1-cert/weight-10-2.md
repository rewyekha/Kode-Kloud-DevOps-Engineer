# Weight: 10

We received a request to copy some of the data from one of the docker containers to the docker host. The container is running on `App Server 2` in `Stratos Datacenter`. Below are more details about the task:

On `App Server 2` in `Stratos Datacenter` copy an encrypted file `/tmp/test.txt.gpg` from `development_3` docker container to the docker host in `/tmp` location. Please do not try to modify this file in any way.

```bash
[steve@stapp02 ~]$ ssh steve@stapp02
steve@stapp02's password: 
Permission denied, please try again.
steve@stapp02's password: 
Last login: Thu Mar 12 08:33:22 2026 from 10.244.195.17
[steve@stapp02 ~]$ sudo docker ps
[sudo] password for steve: 
CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS              PORTS                               NAMES
d8d47918300b   httpd:alpine            "httpd-foreground"       About a minute ago   Up About a minute   80/tcp                              development_3
1878489387e3   ubuntu/apache2:latest   "apache2-foreground"     5 minutes ago        Up 5 minutes        80/tcp                              ubuntu_latest
076462effdb6   nginx:alpine            "/docker-entrypoint.…"   7 minutes ago        Up 7 minutes        0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_2
5bb9a426f29c   ubuntu/apache2:latest   "sleep 1000"             10 minutes ago       Up 10 minutes       80/tcp                              debug_2
[steve@stapp02 ~]$ sudo docker cp development_3:/tmp/test.txt.gpg /tmp/
Successfully copied 2.05kB to /tmp/
[steve@stapp02 ~]$ ls -l /tmp/test.txt.gpg
-rw-r--r-- 1 root root 98 Mar 12 08:35 /tmp/test.txt.gpg
[steve@stapp02 ~]$ 
```
