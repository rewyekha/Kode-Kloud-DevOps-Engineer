# Weight: 10

The Nautilus DevOps team has some confidential data present on `App Server 2` in `Stratos Datacenter`. There is a container `ubuntu_latest` running on the same server. We received a request to copy some of the data from the docker host to the container. Below are more details about the task:

On `App Server 2` in `Stratos Datacenter` copy an encrypted file `/tmp/nautilus.txt.gpg` from docker host to `ubuntu_latest` container (running on same server) in `/home/` location (create this location if doesn't exit). Please do not try to modify this file in any way.

```bash
[steve@stapp02 ~]$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.195.17)' can't be established.
ED25519 key fingerprint is SHA256:ybtZN+ZJADPxpY7GDHZw2XoIgk+FiUQWum9PNMLit3w.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
Last login: Thu Mar 12 08:25:02 2026 from 10.244.49.51
[steve@stapp02 ~]$ sudo docker ps
[sudo] password for steve: 
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                               NAMES
1878489387e3   ubuntu/apache2:latest   "apache2-foreground"     2 minutes ago   Up 2 minutes   80/tcp                              ubuntu_latest
076462effdb6   nginx:alpine            "/docker-entrypoint.…"   3 minutes ago   Up 3 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_2
5bb9a426f29c   ubuntu/apache2:latest   "sleep 1000"             6 minutes ago   Up 6 minutes   80/tcp                              debug_2
[steve@stapp02 ~]$ ls
[steve@stapp02 ~]$ sudo docker exec ubuntu_latest mkdir -p /home
[steve@stapp02 ~]$ sudo docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/home/
Successfully copied 2.05kB to ubuntu_latest:/home/
[steve@stapp02 ~]$ sudo docker exec ubuntu_latest ls -l /home/
total 8
-rw-r--r-- 1 root   root    105 Mar 12 08:31 nautilus.txt.gpg
drwxr-x--- 2 ubuntu ubuntu 4096 Oct  1 02:10 ubuntu
[steve@stapp02 ~]$ 
```
