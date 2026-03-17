# Weight: 10

The DevOps team is performing some cleanup on all app servers in `Stratos DC`. They want to clean up some unwanted docker images from these servers, some images might be in use by some docker containers, but those containers are not in use so we need to clean those containers and the images. Below are the images that need to be deleted from `Application Server 2`:

a. alpine:3.18.4

b. sebp/lighttpd:latest

```bash
[steve@stapp02 ~]$ sudo docker images
REPOSITORY       TAG        IMAGE ID       CREATED         SIZE
alpine           nautilus   0c3615e82705   7 seconds ago   8.44MB
nginx            alpine     d0c780774910   34 hours ago    62.2MB
httpd            alpine     df4281218506   6 weeks ago     67.1MB
alpine           latest     a40c03cbb81c   6 weeks ago     8.44MB
ubuntu/apache2   latest     d9c1fa2be67f   3 months ago    156MB
[steve@stapp02 ~]$ sudo docker images
REPOSITORY       TAG        IMAGE ID       CREATED              SIZE
alpine           nautilus   0c3615e82705   About a minute ago   8.44MB
nginx            alpine     d0c780774910   34 hours ago         62.2MB
httpd            alpine     df4281218506   6 weeks ago          67.1MB
alpine           latest     a40c03cbb81c   6 weeks ago          8.44MB
ubuntu/apache2   latest     d9c1fa2be67f   3 months ago         156MB
alpine           3.18.4     8ca4688f4f35   2 years ago          7.33MB
sebp/lighttpd    latest     fbbc9e0f56f7   3 years ago          9.53MB
[steve@stapp02 ~]$ sudo docker ps -a
CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS                            PORTS                               NAMES
9e74d050a2e8   sebp/lighttpd:latest    "start.sh"               About a minute ago   Exited (255) About a minute ago                                       lighttpd
04da0fa1b010   alpine                  "/bin/sleep 100000"      3 minutes ago        Up 3 minutes                                                          alpine_nautilus
d8d47918300b   httpd:alpine            "httpd-foreground"       6 minutes ago        Up 6 minutes                      80/tcp                              development_3
1878489387e3   ubuntu/apache2:latest   "apache2-foreground"     11 minutes ago       Up 11 minutes                     80/tcp                              ubuntu_latest
076462effdb6   nginx:alpine            "/docker-entrypoint.…"   12 minutes ago       Up 12 minutes                     0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_2
5bb9a426f29c   ubuntu/apache2:latest   "sleep 1000"             15 minutes ago       Up 15 minutes                     80/tcp                              debug_2
[steve@stapp02 ~]$ sudo docker rm 9e74d050a2e8
9e74d050a2e8
[steve@stapp02 ~]$ sudo docker rmi sebp/lighttpd:latest
Untagged: sebp/lighttpd:latest
Untagged: sebp/lighttpd@sha256:43d2a1c6d6a2ef85cc1914daa7dbb16776b2ec8bd0e0e7b824caa9bc5079773c
Deleted: sha256:fbbc9e0f56f7dabf00f0053a26635af09476c6baa0fa120f25044273e8d2b39a
Deleted: sha256:b2d27870bbf07b793e052b009e13a0ce96ad28166e5bf3a08f515019309c4297
Deleted: sha256:ee2ee4421825006b870763bfa63dcada0d9959be8ba4e47d9d8bc598e3965cd3
Deleted: sha256:db639f2ad8b2a4866e264776f72d77cd6c0ea14daf0a62d43a88bb57c76cbb55
Deleted: sha256:24302eb7d9085da80f016e7e4ae55417e412fb7e0a8021e95e3b60c67cde557d
[steve@stapp02 ~]$ sudo docker rmi alpine:3.18.4
Untagged: alpine:3.18.4
Untagged: alpine@sha256:eece025e432126ce23f223450a0326fbebde39cdf496a85d8c016293fc851978
Deleted: sha256:8ca4688f4f356596b5ae539337c9941abc78eda10021d35cbc52659c74d9b443
Deleted: sha256:cc2447e1835a40530975ab80bb1f872fbab0f2a0faecf2ab16fbbb89b3589438
[steve@stapp02 ~]$ sudo docker images
REPOSITORY       TAG        IMAGE ID       CREATED         SIZE
alpine           nautilus   0c3615e82705   3 minutes ago   8.44MB
nginx            alpine     d0c780774910   34 hours ago    62.2MB
httpd            alpine     df4281218506   6 weeks ago     67.1MB
alpine           latest     a40c03cbb81c   6 weeks ago     8.44MB
ubuntu/apache2   latest     d9c1fa2be67f   3 months ago    156MB
[steve@stapp02 ~]$ 
```
