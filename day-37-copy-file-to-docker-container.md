# Day 37: Copy File to Docker Container
## Copy Encrypted File from Docker Host to Running Container (Q\&A Guide)
***

### What is the task?
We need to copy an encrypted file:

```
/tmp/nautilus.txt.gpg
```

From the Docker host (**App Server 2**) into a running container named:

```
ubuntu_latest
```

And place it inside:

```
/home/
```

Without modifying the file.

***

## Step 1: How do we connect to App Server 2?
From the jump host:

```bash
thor@jumphost ~$ ssh steve@stapp02.stratos.xfusioncorp.com
```

***

### What error did we face while connecting?
#### Error: Host Key Verification Failed
```bash
The authenticity of host 'stapp02.stratos.xfusioncorp.com (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:dZUYF5QlkrO/4ju0Rcq3yUi3AezeaSHwCQHaJRVcskE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? no
Host key verification failed.
```

#### Why did this happen?
Because we typed:

```
no
```

SSH could not verify the host key and refused the connection.

***

### How was it fixed?
Reconnect and type:

```bash
yes
```

Successful connection:

```bash
Warning: Permanently added 'stapp02.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
steve@stapp02.stratos.xfusioncorp.com's password:
```

***

## Step 2: How do we verify the container is running?
```bash
[steve@stapp02 ~]$ docker ps
```

#### Output:
```bash
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
9a66e84d7553   ubuntu    "/bin/bash"   About a minute ago   Up About a minute             ubuntu_latest
```

#### What does this confirm?
* Container `ubuntu_latest` is running
* We can safely copy files into it

***

## Step 3: How do we copy the encrypted file?
```bash
[steve@stapp02 ~]$ docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/home/
```

#### Output:
```bash
Successfully copied 2.05kB to ubuntu_latest:/home/
```

#### Does this modify the file?
No.
`docker cp` copies the file as-is without altering contents.

***

## Step 4: How do we verify inside the container?
```bash
[steve@stapp02 ~]$ docker exec -it ubuntu_latest ls -l /home/
```

#### Output:
```bash
total 8
-rw-r--r-- 1 root   root    105 Mar  2 03:56 nautilus.txt.gpg
drwxr-x--- 2 ubuntu ubuntu 4096 Feb 10 14:12 ubuntu
```

#### What does this confirm?
* File exists inside container
* Located at `/home/nautilus.txt.gpg`
* Successfully copied
* No modification occurred

***

## What Common Errors Can Occur?
***

### What if the container name is wrong?
#### Error:
```bash
Error: No such container: ubuntu-latest
```

#### Cause:
Typo in container name.

#### Fix:
Check correct name:

```bash
docker ps
```

***

### What if the container is not running?
#### Issue:
Container does not appear in `docker ps`.

#### Fix:
Start it:

```bash
docker start ubuntu_latest
```

***

### What if the file does not exist on the host?
#### Error:
```bash
Error: stat /tmp/nautilus.txt.gpg: no such file or directory
```

#### Fix:
Verify file:

```bash
ls -l /tmp/nautilus.txt.gpg
```

***

## Final Outcome
SSH connection established
Container verified running
Encrypted file copied successfully
File integrity preserved
Task completed without modification

***

## Command Summary
```bash
ssh steve@stapp02.stratos.xfusioncorp.com
docker ps
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/home/
docker exec -it ubuntu_latest ls -l /home/
```
