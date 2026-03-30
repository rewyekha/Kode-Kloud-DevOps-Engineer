# Delete Docker Container

Here is a GitBook-style documentation entry for deleting the `kke-container` on App Server 3:

***

## Deleting the `kke-container` on App Server 3

### Task Overview

One of the Nautilus project developers created a container named `kke-container` on **App Server 3** (`stapp03`). It was used for testing and now requires deletion.

***

### Step 1: SSH into App Server 3

Connect to the server using the following command:

```bash
ssh banner@172.16.238.12
```

* Accept the authenticity prompt by typing `yes`.
* Enter the password when prompted: `BigGr33n`

***

### Step 2: Verify the Container Status

Check all Docker containers (running and stopped):

```bash
docker ps -a
```

Expected output shows the container `kke-container` running:

```
CONTAINER ID   IMAGE     COMMAND               CREATED              STATUS              PORTS     NAMES
44eb8c81d361   busybox   "tail -f /dev/null"   About a minute ago   Up About a minute             kke-container
```

***

### Step 3: Stop the Container

Stop the running container:

```bash
docker stop kke-container
```

Verify that the container has stopped:

```bash
docker ps -a
```

Expected output shows the container status as exited:

```
CONTAINER ID   IMAGE     COMMAND               CREATED         STATUS                       PORTS     NAMES
44eb8c81d361   busybox   "tail -f /dev/null"   2 minutes ago   Exited (137) 6 seconds ago             kke-container
```

***

### Step 4: Remove the Container with Sudo

Since Docker requires elevated privileges, use `sudo` to remove the container:

```bash
sudo docker rm kke-container
```

Enter the password `BigGr33n` when prompted.

***

### Step 5: Confirm Container Removal

Run the following command to ensure the container no longer exists:

```bash
docker ps -a
```

Expected output:

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

No containers named `kke-container` should appear.

***

### Summary

* SSH into App Server 3
* List Docker containers to confirm presence of `kke-container`
* Stop the container
* Remove the container using `sudo`
* Confirm the container is deleted

***

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
