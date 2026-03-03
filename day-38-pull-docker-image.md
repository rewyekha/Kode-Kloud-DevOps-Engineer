# Day 38: Pull Docker Image

Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:

a. Pull `busybox:musl` image on `App Server 2` in Stratos DC and re-tag (create new tag) this image as `busybox:blog`.



## Pull and Re-Tag Docker Image (busybox:musl → busybox:blog)

### 📌 Objective

Connect to **App Server 2 (stapp02)** via SSH and:

1. Pull `busybox:musl` image from Docker Hub
2. Re-tag it as `busybox:blog`
3. Verify the new tag

***

## 🔐 Step 1: SSH into App Server 2

### ❌ Common Mistake (Wrong SSH Syntax)

```bash
thor@jumphost ~$ ssh steve@     172.16.238.11
ssh: Could not resolve hostname : Name or service not known
```

#### 🔎 Why This Happens?

There is an unintended space between `steve@` and the IP address.

**SSH syntax must be:**

```bash
ssh user@host
```

If a space is added, SSH treats the host as empty → resulting in hostname resolution error.

***

### ✅ Correct SSH Command

```bash
thor@jumphost ~$ ssh steve@172.16.238.11
```

#### First-Time Connection Output

```bash
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:yAAV75PzPb7MoqoD0ejf4nbs4aGiM6j1ByU6ZiS+YnM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password:
```

#### 🔎 What This Means

* SSH does not recognize the remote server.
* It asks for confirmation to store the server's public key.
* Typing `yes` adds it to:

```
~/.ssh/known_hosts
```

#### 🟢 Successful Login

```bash
[steve@stapp02 ~]$
```

You are now logged into **App Server 2**.

***

## 🐳 Step 2: Pull the Docker Image

### Command

```bash
docker pull busybox:musl
```

### Terminal Output

```bash
musl: Pulling from library/busybox
5bfa213ad291: Pull complete 
Digest: sha256:19b646668802469d968a05342a601e78da4322a414a7c09b1c9ee25165042138
Status: Downloaded newer image for busybox:musl
docker.io/library/busybox:musl
```

### 🔎 Command Explanation

| Part          | Meaning                              |
| ------------- | ------------------------------------ |
| `docker pull` | Downloads image from Docker registry |
| `busybox`     | Image name                           |
| `musl`        | Tag/version of the image             |
| `library/`    | Official Docker Hub repository       |

***

## 🔍 Step 3: Verify the Image

```bash
docker images | grep busybox
```

### Output

```bash
busybox      musl      0188a8de47ca   17 months ago   1.51MB
```

### 🔎 Explanation

* `docker images` → Lists all images
* `grep busybox` → Filters only busybox images
* `0188a8de47ca` → Image ID
* Size is `1.51MB`

***

## 🏷️ Step 4: Re-Tag the Image

### Command

```bash
docker tag busybox:musl busybox:blog
```

### 🔎 What This Does

Creates a **new tag** (`blog`) pointing to the same image ID.

> Important: Docker does NOT duplicate the image.\
> It only creates another reference to the same image.

***

## 🔍 Step 5: Verify New Tag

```bash
docker images | grep busybox
```

### Output

```bash
busybox      blog      0188a8de47ca   17 months ago   1.51MB
busybox      musl      0188a8de47ca   17 months ago   1.51MB
```

#### ✅ Observation

Both tags point to:

```
0188a8de47ca
```

Meaning:

* Same image
* Two different tags
* No extra disk usage

***

## 🚨 Common Issues & Troubleshooting

### 1️⃣ SSH: Permission Denied

```bash
Permission denied (publickey,password).
```

#### Causes:

* Wrong password
* SSH key not configured
* User not allowed on server

***

### 2️⃣ Docker Command Not Found

```bash
docker: command not found
```

#### Causes:

* Docker not installed
* User not in docker group

Fix (if needed):

```bash
sudo usermod -aG docker steve
```

***

### 3️⃣ Permission Denied While Running Docker

```bash
Got permission denied while trying to connect to the Docker daemon
```

#### Fix:

```bash
sudo docker pull busybox:musl
```

Or add user to docker group.

***

### 4️⃣ Image Not Found Error

```bash
Error response from daemon: manifest for busybox:musl not found
```

#### Causes:

* Wrong tag name
* Network issue
* Docker Hub access blocked

***

## 📚 Summary

| Step         | Command                                |
| ------------ | -------------------------------------- |
| SSH Login    | `ssh steve@172.16.238.11`              |
| Pull Image   | `docker pull busybox:musl`             |
| Verify Image | \`docker images                        |
| Re-tag Image | `docker tag busybox:musl busybox:blog` |
| Verify Again | \`docker images                        |

***

## ✅ Final Result

You have successfully:

* Connected to App Server 2
* Pulled `busybox:musl`
* Created a new tag `busybox:blog`
* Verified both tags point to the same image



<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
