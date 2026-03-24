# Pull Docker Image

Nautilus project developers are planning to start testing on a new project. As per their meeting with the DevOps team, they want to test containerized environment application features. As per details shared with DevOps team, we need to accomplish the following task:

a. Pull `busybox:musl` image on `App Server 2` in Stratos DC and re-tag (create new tag) this image as `busybox:blog`.



***

## Container Image Preparation Guide

### Objective

As part of the Nautilus project testing requirements, the DevOps team must prepare a container image on **App Server 2** in the Stratos Data Center.

The task involves:

* Pulling the `busybox:musl` image from Docker Hub
* Re-tagging the image as `busybox:blog`

***

### Prerequisites

* SSH access to **App Server 2 (`stapp02`)**
* Valid user credentials
* Docker installed and running on the target server

***

### Procedure

#### 1. Connect to App Server 2

Log in to the server using SSH:

```bash
ssh steve@stapp02
```

> Accept the host authenticity prompt if connecting for the first time.

***

#### 2. Pull the Required Docker Image

Download the `busybox:musl` image from Docker Hub:

```bash
docker pull busybox:musl
```

**Expected Output (sample):**

```
musl: Pulling from library/busybox
Status: Downloaded newer image for busybox:musl
```

***

#### 3. Re-tag the Docker Image

Create a new tag `busybox:blog` from the pulled image:

```bash
docker image tag busybox:musl busybox:blog
```

***

#### 4. Verify the Image Tags

Confirm that both image tags exist:

```bash
docker images | grep busybox
```

**Expected Output:**

```
busybox      blog      <IMAGE_ID>   <CREATED>   <SIZE>
busybox      musl      <IMAGE_ID>   <CREATED>   <SIZE>
```

> Note: Both tags should reference the same IMAGE ID, confirming successful re-tagging.

***

### Result

* The `busybox:musl` image has been successfully pulled.
* A new tag `busybox:blog` has been created.
* Both tags are available locally on **App Server 2**.

***

### Notes

* Re-tagging does not duplicate the image; it only creates an additional reference.
* This setup enables testing workflows using a custom-tagged container image.

***

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
