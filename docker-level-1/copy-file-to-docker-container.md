# Copy File to Docker Container

The Nautilus DevOps team possesses confidential data on App Server 3 in the Stratos Datacenter. A container named ubuntu\_latest is running on the same server. Copy an encrypted file /tmp/nautilus.txt.gpg from the docker host to the ubuntu\_latest container located at /usr/src/. Ensure the file is not modified during this operation.

Since the container **ubuntu\_latest** is running on **stapp03 (App Server 3)**, you need to copy the file from the host to the container using `docker cp`.

#### ✅ Steps

1. **SSH into App Server 3**

```bash
ssh banner@stapp03.stratos.xfusioncorp.com
# Password: BigGr33n
```

2. **Verify the container is running**

```bash
docker ps
```

3. **Copy the encrypted file to the container**

```bash
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
```

#### 🔎 Explanation

* `docker cp` copies files between host and container.
* `/tmp/nautilus.txt.gpg` → Source file on host.
* `ubuntu_latest:/usr/src/` → Destination inside container.
* This method preserves the file exactly as-is (no modification).

4. **(Optional) Verify inside container**

```bash
docker exec -it ubuntu_latest ls -l /usr/src/
```

You should see:

```
nautilus.txt.gpg
```

✔ The encrypted file is now successfully copied to `/usr/src/` inside the `ubuntu_latest` container without modification.
