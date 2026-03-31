# Docker Node App

## Docker: Dockerize and Deploy a Node.js Web Application

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Beginner | **Topic:** Docker, Node.js, Dockerfile, Express, Container Deployment

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Solution](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#solution)
   * [Step 1: SSH into Application Server 2](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-2)
   * [Step 2: Switch to Root](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-switch-to-root)
   * [Step 3: Inspect Existing Application Files](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-inspect-existing-application-files)
   * [Step 4: Create the Dockerfile](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-create-the-dockerfile)
   * [Step 5: Build the Docker Image](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-build-the-docker-image)
   * [Step 6: Run the Container](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-run-the-container)
   * [Step 7: Verify and Test](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-verify-and-test)
4. [Final Dockerfile](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#final-dockerfile)
5. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
6. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

There is a requirement to Dockerize a Node app and deploy it on **App Server 2**. Under `/node_app` directory on App Server 2, a `package.json` file and `server.js` file have already been placed.

**Requirements:**

1. Create a `Dockerfile` (name is case sensitive) under `/node_app` directory:
   * Use any `node` image as the base image
   * Install the dependencies using `package.json` file
   * Use `server.js` in the `CMD`
   * Expose port `5003`
2. The built image should be named `nautilus/node-web-app`
3. Run a container named `nodeapp_nautilus` using this image:
   * Map container port `5003` with host port `8095`
4.  Test the deployed app:

    ```bash
    curl http://localhost:8095
    ```

***

### Infrastructure Details

> **Target:** Application Server 2 (`stapp02`) — all work is performed on this server.

***

### Solution

#### Step 1: SSH into Application Server 2

From the jump host, connect to `stapp02` as user `steve`:

```bash
ssh steve@stapp02
```

**Terminal Output:**

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.97.181)' can't be established.
ED25519 key fingerprint is SHA256:VoQfU3ikTU7EgUyX20pohnP6r04vXpG1iSPYoiwgOfY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password:
[steve@stapp02 ~]$
```

Enter password: `Am3ric@`

***

#### Step 2: Switch to Root

```bash
sudo su -
```

**Terminal Output:**

```bash
[steve@stapp02 ~]$ sudo su -

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve:
[root@stapp02 ~]#
```

Enter password: `Am3ric@`

***

#### Step 3: Inspect Existing Application Files

Check what files are present in the `/node_app` directory and review their contents:

```bash
ls /node_app/
```

**Terminal Output:**

```bash
[root@stapp02 ~]# ls /node_app/
package.json  server.js
```

Review the `package.json` to understand app dependencies:

```bash
cat /node_app/package.json
```

**Terminal Output:**

```bash
[root@stapp02 ~]# cat /node_app/package.json
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "Sample Test <sample.test@example.com>",
  "main": "server.js",
  "keywords": [
    "nodejs",
    "bootstrap",
    "express"
  ],
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

Review the `server.js` to understand the app behaviour:

```bash
cat /node_app/server.js
```

**Terminal Output:**

```bash
[root@stapp02 ~]# cat /node_app/server.js
'use strict';

const express = require('express');

// Constants
const PORT = 5003;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Welcome to xFusionCorp Industries!');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

The app is an Express.js web server that:

* Listens on port `5003` on all interfaces (`0.0.0.0`)
* Responds to `GET /` with `Welcome to xFusionCorp Industries!`
* Has a single dependency: `express ^4.16.1`

***

#### Step 4: Create the Dockerfile

Create the `Dockerfile` inside `/node_app`:

```bash
vi /node_app/Dockerfile
```

Enter the following content:

```dockerfile
FROM node:latest
WORKDIR /app
COPY package.json .
RUN npm install
COPY server.js .
EXPOSE 5003
CMD ["node", "server.js"]
```

Save and exit: `Esc` → `:wq` → `Enter`

Verify the file was written correctly:

```bash
cat /node_app/Dockerfile
```

**Terminal Output:**

```bash
[root@stapp02 ~]# cat /node_app/Dockerfile
FROM node:latest
WORKDIR /app
COPY package.json .
RUN npm install
COPY server.js .
EXPOSE 5003
CMD ["node", "server.js"]
```

***

#### Step 5: Build the Docker Image

Navigate to the app directory and build the image:

```bash
cd /node_app
docker build -t nautilus/node-web-app .
```

**Terminal Output:**

```bash
[root@stapp02 node_app]# docker build -t nautilus/node-web-app .
[+] Building 38.4s (10/10) FINISHED                        docker:default
 => [internal] load build definition from Dockerfile                 0.1s
 => => transferring dockerfile: 159B                                 0.0s
 => [internal] load metadata for docker.io/library/node:latest       1.5s
 => [internal] load .dockerignore                                    0.1s
 => => transferring context: 2B                                      0.0s
 => [1/5] FROM docker.io/library/node:latest@sha256:ccfc02deb6abb1  21.6s
 => => resolve docker.io/library/node:latest@sha256:ccfc02deb6abb1b  0.1s
 => => sha256:6cf051f1897bf7109af670b243c7791c627 64.40MB / 64.40MB  2.2s
 => => sha256:9d2f29087bcd6d99efd909a99095549425c 48.49MB / 48.49MB  1.7s
 => => sha256:26fa3468d221545a43d2151f3977695a318 24.04MB / 24.04MB  1.2s
 => => sha256:505954b662451663b30768d461d6881e4 211.53MB / 211.53MB  5.1s
 => => sha256:aae062f973ebdbee978215c3f21df80bbc0 57.67MB / 57.67MB  3.9s
 => => sha256:5e41a256126a2332261ce2928c4ae0c38ba40 1.25MB / 1.25MB  2.6s
 => => extracting sha256:9d2f29087bcd6d99efd909a99095549425cd63e27c  2.1s
 => => extracting sha256:26fa3468d221545a43d2151f3977695a31857f9342  1.6s
 => => extracting sha256:6cf051f1897bf7109af670b243c7791c62723fc1eb  3.0s
 => => extracting sha256:505954b662451663b30768d461d6881e47bb272f6b  8.2s
 => => extracting sha256:aae062f973ebdbee978215c3f21df80bbc02ae2878  2.2s
 => => extracting sha256:5e41a256126a2332261ce2928c4ae0c38ba409c552  0.1s
 => [internal] load build context                                    0.1s
 => => transferring context: 709B                                    0.0s
 => [2/5] WORKDIR /app                                               0.1s
 => [3/5] COPY package.json .                                        0.1s
 => [4/5] RUN npm install                                            9.4s
 => [5/5] COPY server.js .                                           0.2s
 => exporting to image                                               5.1s
 => => exporting layers                                              5.0s
 => => writing image sha256:0b79624766600b0e62a5e02bde0d53c0bb955b7  0.0s
 => => naming to docker.io/nautilus/node-web-app                     0.0s
```

The build completed in `38.4s` across 10 steps — all 5 Dockerfile instructions executed successfully and the image was tagged as `nautilus/node-web-app`.

***

#### Step 6: Run the Container

Start the container in detached mode with the required port mapping:

```bash
docker run -d \
  --name nodeapp_nautilus \
  -p 8095:5003 \
  nautilus/node-web-app
```

**Terminal Output:**

```bash
[root@stapp02 node_app]# docker run -d \
  --name nodeapp_nautilus \
  -p 8095:5003 \
  nautilus/node-web-app
2e45bdccbb7fd60e9d44d31c66fd14177ab6f083ca8a6667391546be59baf069
```

Docker returned the full container ID `2e45bdccbb7fd60e9d44d31c66fd14177ab6f083ca8a6667391546be59baf069`, confirming successful creation.

***

#### Step 7: Verify and Test

Confirm the container is running:

```bash
docker ps
```

**Terminal Output:**

```
[root@stapp02 node_app]# docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS         PORTS                                       NAMES
2e45bdccbb7f   nautilus/node-web-app   "docker-entrypoint.s…"   5 seconds ago   Up 3 seconds   0.0.0.0:8095->5003/tcp, :::8095->5003/tcp   nodeapp_nautilus
```

Test the web application with `curl`:

```bash
curl http://localhost:8095
```

**Terminal Output:**

```
[root@stapp02 node_app]# curl http://localhost:8095
Welcome to xFusionCorp Industries!
```

The Node.js Express app is running and responding correctly on host port `8095`.

***

### Final Dockerfile

```dockerfile
FROM node:latest
WORKDIR /app
COPY package.json .
RUN npm install
COPY server.js .
EXPOSE 5003
CMD ["node", "server.js"]
```

***

### Lab Complete

| Requirement            | Detail                                    | Status    |
| ---------------------- | ----------------------------------------- | --------- |
| Server                 | `stapp02`                                 | Confirmed |
| Dockerfile location    | `/node_app/Dockerfile`                    | Confirmed |
| Base image             | `node:latest`                             | Confirmed |
| Dependencies installed | `npm install` from `package.json`         | Confirmed |
| Exposed port           | `5003`                                    | Confirmed |
| Image name             | `nautilus/node-web-app`                   | Confirmed |
| Image SHA              | `0b79624766600b0e62a5e02bde0d53c0bb955b7` | Confirmed |
| Container name         | `nodeapp_nautilus`                        | Confirmed |
| Port mapping           | `0.0.0.0:8095->5003/tcp`                  | Confirmed |
| Container status       | `Up 3 seconds`                            | Confirmed |
| curl response          | `Welcome to xFusionCorp Industries!`      | Confirmed |

***

### Key Concepts

#### Dockerfile Instruction Order and Layer Caching

The Dockerfile was written in a specific order to maximise Docker layer caching efficiency:

```docker
COPY package.json .      # Step 1 — copy only package.json first
RUN npm install          # Step 2 — install dependencies (cached if package.json unchanged)
COPY server.js .         # Step 3 — copy app code last
```

Copying `package.json` and running `npm install` before copying `server.js` is a best practice. If only `server.js` changes, Docker reuses the cached `npm install` layer — saving significant build time. If `package.json` and `server.js` were copied together in one instruction, any code change would invalidate the install cache and force a full `npm install` on every build.

#### Port Mapping: Host vs Container

```bash
curl http://localhost:8095
         │
         │ Host port 8095
         ▼
Docker NAT (0.0.0.0:8095->5003/tcp)
         │
         │ Container port 5003
         ▼
Express app listening on 0.0.0.0:5003 inside container
```

| Port   | Location         | Set By                                                          |
| ------ | ---------------- | --------------------------------------------------------------- |
| `5003` | Inside container | `EXPOSE 5003` in Dockerfile and `app.listen(PORT)` in server.js |
| `8095` | On the host      | `-p 8095:5003` in `docker run`                                  |

#### `WORKDIR` in Docker

The `WORKDIR /app` instruction sets the working directory inside the container for all subsequent `COPY`, `RUN`, and `CMD` instructions. It also creates the directory if it does not exist. Using a dedicated working directory like `/app` rather than root `/` is a security and organisation best practice — it keeps application files isolated from system files.

#### `CMD` vs `ENTRYPOINT`

```dockerfile
CMD ["node", "server.js"]
```

`CMD` specifies the default command to run when the container starts. Using the **exec form** (`["node", "server.js"]`) rather than the **shell form** (`node server.js`) is preferred because:

* It runs `node` directly as PID 1 — not wrapped in a shell
* Signals (like `SIGTERM` from `docker stop`) are received directly by the Node process
* Cleaner and more predictable shutdown behaviour

***

_Lab completed on 2026-03-31 | Server: stapp02 | OS: CentOS Stream 9 | Docker: default_

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
