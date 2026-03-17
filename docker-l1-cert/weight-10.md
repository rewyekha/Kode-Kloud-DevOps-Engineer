# Weight: 10

The Nautilus DevOps team is testing some applications deployment on some of the application servers. They need to deploy a nginx container on `Application Server 2`. Please complete the task as per details given below:

On `Application Server 2` create a container named `nginx_2` using image `nginx` with `alpine` tag and make sure container is in `running` state.

***

**1. SSH into Application Server 2**

```bash
ssh steve@stapp02.stratos.xfusioncorp.com
```

Password: `Am3ric@`

***

**2. Pull the nginx image with Alpine tag**

```bash
sudo docker pull nginx:alpine
```

***

**3. Create and run the container**

```bash
sudo docker run -d --name nginx_2 -p 80:80 nginx:alpine
```

Explanation:

* `-d` → Run in detached mode.
* `--name nginx_2` → Container name.
* `-p 80:80` → Expose port 80 (optional if you want to access externally).
* `nginx:alpine` → Use nginx with the Alpine Linux tag.

***

**4. Verify the container is running**

```bash
sudo docker ps
```

You should see `nginx_2` in the running state with the image `nginx:alpine`.

```bash
[steve@stapp02 ~]$ sudo docker pull nginx:alpine
alpine: Pulling from library/nginx
589002ba0eae: Pull complete 
d2a46166eee6: Pull complete 
593488f95c35: Pull complete 
e19aff8f2cce: Pull complete 
1549d7aec962: Pull complete 
1f25242adbdb: Pull complete 
c32126d2b96c: Pull complete 
c24026275c33: Pull complete 
Digest: sha256:f46cb72c7df02710e693e863a983ac42f6a9579058a59a35f1ae36c9958e4ce0
Status: Downloaded newer image for nginx:alpine
docker.io/library/nginx:alpine
[steve@stapp02 ~]$ sudo docker run -d --name nginx_2 -p 80:80 nginx:alpine
076462effdb6a3d418c028ad183fae12e5fa1e75a600dee381af2abc37c069e3
[steve@stapp02 ~]$ sudo docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS         PORTS                               NAMES
076462effdb6   nginx:alpine            "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   nginx_2
5bb9a426f29c   ubuntu/apache2:latest   "sleep 1000"             3 minutes ago    Up 3 minutes   80/tcp                              debug_2
[steve@stapp02 ~]$ 
```
