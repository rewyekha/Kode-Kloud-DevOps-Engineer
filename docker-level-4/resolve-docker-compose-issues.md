# Resolve Docker Compose Issues

The Nautilus DevOps team is working to deploy one of the applications on `App Server 1` in `Stratos DC`. Due to a misconfiguration in the docker compose file, the deployment is failing. We would like you to take a look into it to identify and fix the issues. More details can be found below:

a. `docker-compose.yml` file is present on `App Server 1` under `/opt/docker` directory.

b. Try to run the same and make sure it works fine.

c. Please do not change the `container names` being used. Also, do not update or alter any other valid config settings in the compose file or any other relevant data that can cause app failure.

`Note:` Please note that once you click on `FINISH` button all existing running/stopped containers will be destroyed, and your compose will be run.



## Docker Compose: Troubleshoot and Fix Misconfigured Deployment

> **Platform:** KodeKloud | **Series:** Nautilus DevOps — Stratos Datacenter **Difficulty:** Intermediate | **Topic:** Docker Compose, Troubleshooting, Python Flask, Redis

***

### Table of Contents

1. [Lab Question](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-question)
2. [Infrastructure Details](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#infrastructure-details)
3. [Investigation](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#investigation)
   * [Step 1: SSH into Application Server 1](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-1-ssh-into-application-server-1)
   * [Step 2: Inspect the Directory Structure](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-2-inspect-the-directory-structure)
   * [Step 3: Inspect the Dockerfile](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-3-inspect-the-dockerfile)
   * [Step 4: Inspect the Original docker-compose.yml](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-4-inspect-the-original-docker-composeyml)
4. [Root Cause Analysis](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#root-cause-analysis)
5. [Fix](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#fix)
   * [Step 5: First Fix — Correct redis key and build path](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-5-first-fix--correct-redis-key-and-build-path)
   * [Step 6: Second Fix — Correct the volume mount path](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-6-second-fix--correct-the-volume-mount-path)
   * [Step 7: Clean Up Conflicting Containers](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-7-clean-up-conflicting-containers)
   * [Step 8: Final Deploy](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-8-final-deploy)
   * [Step 9: Verify Both Containers are Running](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-9-verify-both-containers-are-running)
   * [Step 10: Test the Web Application](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#step-10-test-the-web-application)
6. [Final docker-compose.yml](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#final-docker-composeyml)
7. [Lab Complete](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#lab-complete)
8. [Key Concepts](https://claude.ai/chat/2167892e-87fd-41a5-a1b7-351d743253cc#key-concepts)

***

### Lab Question

The Nautilus DevOps team is working to deploy one of the applications on **App Server 1** in Stratos DC. Due to a misconfiguration in the docker compose file, the deployment is failing.

**Requirements:**

* **a.** `docker-compose.yml` file is present on App Server 1 under `/opt/docker` directory.
* **b.** Try to run the same and make sure it works fine.
* **c.** Do not change the container names being used. Do not update or alter any other valid config settings in the compose file or any other relevant data that can cause app failure.

> **Note:** Once you click on the FINISH button all existing running/stopped containers will be destroyed, and your compose will be run fresh.

***

### Infrastructure Details

| Server Name          | Hostname    | User      | Password     | Purpose                               |
| -------------------- | ----------- | --------- | ------------ | ------------------------------------- |
| Application Server 1 | `stapp01`   | `tony`    | `Ir0nM@n`    | Hosts Nautilus Application 1          |
| Application Server 2 | `stapp02`   | `steve`   | `Am3ric@`    | Hosts Nautilus Application 2          |
| Application Server 3 | `stapp03`   | `banner`  | `BigGr33n`   | Hosts Nautilus Application 3          |
| LoadBalancer Server  | `stlb01`    | `loki`    | `Mischi3f`   | Distributes traffic for Nautilus HTTP |
| Database Server      | `stdb01`    | `peter`   | `Sp!dy`      | Hosts Nautilus Database               |
| Storage Server       | `ststor01`  | `natasha` | `Bl@kW`      | Stores data for Nautilus Servers      |
| Backup Server        | `stbkp01`   | `clint`   | `H@wk3y3`    | Manages backups for Nautilus Servers  |
| Mail Server          | `stmail01`  | `groot`   | `Gr00T123`   | Manages email services                |
| Jump Host            | `jump-host` | `thor`    | `mjolnir123` | Provides secure access to Stork DC    |
| Jenkins Server       | `jenkins`   | `jenkins` | `j@rv!s`     | Runs Jenkins for CI/CD pipeline       |

> **Target:** Application Server 1 (`stapp01`) — fix the docker-compose.yml and get both containers running.

***

### Investigation

#### Step 1: SSH into Application Server 1

```bash
ssh tony@stapp01
```

**Terminal Output:**

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.49.63)' can't be established.
ED25519 key fingerprint is SHA256:nQfrUPRj6ohFxy5y0E5tsjMgSzzfeSBPsJr9ZXRWiHE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$
```

Enter password: `Ir0nM@n`

***

#### Step 2: Inspect the Directory Structure

Navigate to the docker directory and examine what is present:

```bash
cd /opt/docker/
ls
```

**Terminal Output:**

```
[tony@stapp01 ~]$ cd /opt/docker/
[tony@stapp01 docker]$ ls
app  docker-compose.yml
```

```bash
cd app
ls
```

**Terminal Output:**

```
[tony@stapp01 docker]$ cd app
[tony@stapp01 app]$ ls
Dockerfile  app.py  requirements.txt
```

The directory structure is:

```bash
/opt/docker/
├── docker-compose.yml
└── app/
    ├── Dockerfile
    ├── app.py
    └── requirements.txt
```

***

#### Step 3: Inspect the Dockerfile

```bash
cat Dockerfile
```

**Terminal Output:**

```bash
[tony@stapp01 app]$ cat Dockerfile
FROM python:3.13.0b1-slim-bullseye
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD python app.py
```

The Dockerfile copies the app contents to `/code` inside the image and runs `app.py` from there. This is correct and requires no changes.

***

#### Step 4: Inspect the Original docker-compose.yml

Navigate back and read the original compose file:

```bash
cd ..
cat docker-compose.yml
```

**Terminal Output:**

```bash
[tony@stapp01 app]$ cd ..
[tony@stapp01 docker]$ cat docker-compose.yml
name: myapp

service:
    web:
        build: /app
        container_name: python
        ports:
            - "5000:5000"
        volumes:
            - .:/code
        depends_on:
            - redis
    redis:
        from: redis
        container_name: redis
```

Three bugs are immediately visible in this file.

***

### Root Cause Analysis

The original `docker-compose.yml` contained three separate errors that prevented the application from running:

| # | Field                | Bug                                                         | Correct Value            | Impact                                                              |
| - | -------------------- | ----------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------- |
| 1 | `service:`           | Wrong key — should be `services:` (plural)                  | `services:`              | Docker Compose did not recognise any services                       |
| 2 | `build: /app`        | Absolute path `/app` does not exist — the app is at `./app` | `build: ./app`           | `unable to prepare context: path "/app" not found`                  |
| 3 | `from: redis`        | Invalid key — Docker Compose uses `image:` not `from:`      | `image: redis`           | Redis container could not be created                                |
| 4 | `volumes: - .:/code` | Mounts `/opt/docker` over `/code`, hiding `app.py`          | `volumes: - ./app:/code` | `python: can't open file '/code/app.py': No such file or directory` |

The volume mount bug was the most subtle — even after fixing the other three errors, the app still exited because mounting `.` (the `/opt/docker` directory) over `/code` inside the container overwrote the directory where `app.py` was copied during the Docker build. The correct mount is `./app:/code` which maps the actual application directory into the container.

***

### Fix

#### Step 5: First Fix — Correct redis key and build path

The first edit corrected `from: redis` → `image: redis` and `build: /app` → `build: /app` (partial fix, absolute path issue persisted temporarily). After this first edit the `service:` key was also corrected to `services:`.

Running `docker compose up -d` after this partial fix produced:

```
unable to prepare context: path "/app" not found
```

Confirming the absolute build path was still wrong.

***

#### Step 6: Second Fix — Correct the volume mount path

The final correct `docker-compose.yml` was written with all four fixes applied:

```bash
sudo nano /opt/docker/docker-compose.yml
```

Changes made:

```yaml
# Fix 1: service → services
# Fix 2: build: /app → build: ./app
# Fix 3: from: redis → image: redis
# Fix 4: volumes: - .:/code → volumes: - ./app:/code
```

***

#### Step 7: Clean Up Conflicting Containers

During the iterative fixing process, leftover containers from previous failed attempts caused name conflicts. They were removed:

```bash
docker rm -f python redis
docker compose down
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ docker rm -f python redis
python
redis
[tony@stapp01 docker]$ docker compose down
[+] down 1/1
 ✔ Network docker_default Removed                                                                                      0.1s
```

***

#### Step 8: Final Deploy

```bash
docker compose up -d --build
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ docker compose up -d --build
[+] Building 1.0s (11/11) FINISHED
 => [internal] load local bake definitions                                                                            0.0s
 => => reading from stdin 468B                                                                                        0.0s
 => [internal] load build definition from Dockerfile                                                                  0.1s
 => => transferring dockerfile: 151B                                                                                  0.0s
 => [internal] load metadata for docker.io/library/python:3.13.0b1-slim-bullseye                                      0.6s
 => [internal] load .dockerignore                                                                                     0.0s
 => => transferring context: 2B                                                                                       0.0s
 => [internal] load build context                                                                                     0.0s
 => => transferring context: 92B                                                                                      0.0s
 => [1/4] FROM docker.io/library/python:3.13.0b1-slim-bullseye@sha256:6efce108697ffabf20924c157d5f08bc41550aca27a04d  0.0s
 => CACHED [2/4] ADD . /code                                                                                          0.0s
 => CACHED [3/4] WORKDIR /code                                                                                        0.0s
 => CACHED [4/4] RUN pip install -r requirements.txt                                                                  0.0s
 => exporting to image                                                                                                0.0s
 => => exporting layers                                                                                               0.0s
 => => writing image sha256:9ab9bf82f00a58d00c0c8df034427e321af39a740ca2213b58897f7aeb6f7150                          0.0s
 => => naming to docker.io/library/docker-web                                                                         0.0s
 => resolving provenance for metadata file                                                                            0.0s
[+] up 4/4
 ✔ Image docker-web       Built                                                                                        1.1s
 ✔ Network docker_default Created                                                                                      0.1s
 ✔ Container redis        Created                                                                                      0.2s
 ✔ Container python       Created                                                                                      0.1s
```

All four resources created successfully — image built, network created, both containers created.

***

#### Step 9: Verify Both Containers are Running

```bash
docker ps
```

**Terminal Output:**

```bash
[tony@stapp01 docker]$ docker ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                       NAMES
6c3c182295c5   docker-web   "/bin/sh -c 'python …"   5 seconds ago   Up 4 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   python
670253460a14   redis        "docker-entrypoint.s…"   5 seconds ago   Up 4 seconds   6379/tcp                                    redis
```

Both containers are `Up`:

* `python` — Flask web app running on `0.0.0.0:5000->5000/tcp`
* `redis` — Redis cache running on `6379/tcp`

***

#### Step 10: Test the Web Application

Send a request to the Flask application to confirm it is serving correctly and connecting to Redis:

```bash
curl http://localhost:5000
```

**Terminal Output:**

```
[tony@stapp01 docker]$ curl http://localhost:5000
This Compose/Flask demo has been viewed b'1' time(s).[tony@stapp01 docker]$
```

The response `This Compose/Flask demo has been viewed b'1' time(s).` confirms:

* The Flask web server is running and responding on port 5000
* The view counter is being stored and retrieved from Redis successfully
* The full application stack is working end to end

***

### Final docker-compose.yml

The corrected compose file after all fixes:

```yaml
services:
  web:
    build: ./app
    container_name: python
    ports:
      - "5000:5000"
    volumes:
      - ./app:/code
    depends_on:
      - redis

  redis:
    image: redis
    container_name: redis
```

***

### Lab Complete

| Bug                | Original (Broken) | Fixed                    | Status    |
| ------------------ | ----------------- | ------------------------ | --------- |
| Top-level key      | `service:`        | `services:`              | Fixed     |
| Build context path | `build: /app`     | `build: ./app`           | Fixed     |
| Redis image key    | `from: redis`     | `image: redis`           | Fixed     |
| Volume mount path  | `- .:/code`       | `- ./app:/code`          | Fixed     |
| Container `python` | `Exited (2)`      | `Up`                     | Confirmed |
| Container `redis`  | Not created       | `Up`                     | Confirmed |
| Port `5000`        | Not accessible    | `0.0.0.0:5000->5000/tcp` | Confirmed |
| Web response       | Failing           | `viewed b'1' time(s).`   | Confirmed |

***

### Key Concepts

#### Why `service:` vs `services:` Matters

Docker Compose strictly validates the YAML schema. The top-level key `services:` (plural) is required — using `service:` (singular) causes Compose to silently ignore all service definitions, resulting in no containers being created. This is a silent failure that produces no obvious error message.

#### Absolute vs Relative Build Path

```yaml
build: /app      # absolute — looks for /app on the filesystem root
build: ./app     # relative — looks for ./app relative to docker-compose.yml location
```

The compose file lives at `/opt/docker/docker-compose.yml`, so `./app` resolves to `/opt/docker/app` where the actual Dockerfile is located. Using `/app` caused Docker to look for a directory at the root of the filesystem, which does not exist — producing `unable to prepare context: path "/app" not found`.

#### `from:` vs `image:` in Docker Compose

`from:` is not a valid Docker Compose key. The correct key to specify a pre-built image for a service is `image:`. This is likely confused with Dockerfile syntax (`FROM ubuntu:latest`) which uses `FROM` — but in `docker-compose.yml` the correct key is `image:`.

| Context              | Keyword  | Example                              |
| -------------------- | -------- | ------------------------------------ |
| `Dockerfile`         | `FROM`   | `FROM python:3.13.0b1-slim-bullseye` |
| `docker-compose.yml` | `image:` | `image: redis`                       |

#### The Volume Mount Overwrite Problem

This was the most subtle bug. The Dockerfile copied `app.py` into `/code` inside the image:

```dockerfile
ADD . /code        # copies Dockerfile, app.py, requirements.txt into /code
WORKDIR /code
CMD python app.py  # expects app.py to be at /code/app.py
```

But the volume mount `.:/code` then mounted the host directory `/opt/docker` (which contains only `docker-compose.yml` and the `app/` subdirectory — not `app.py` directly) over the container's `/code`, effectively hiding the `app.py` that was placed there during the build.

```bash
Before volume mount:    /code/app.py        ← copied by Dockerfile ADD
After volume mount:     /code/               ← now shows /opt/docker contents
                        /code/app/          ← subdirectory
                        /code/docker-compose.yml
                        (app.py is gone)
```

The fix `./app:/code` mounts the correct host directory — the one that actually contains `app.py` — into `/code` inside the container.

#### `docker logs <container>` for Diagnosing Exit Failures

When a container exits immediately after creation, `docker logs` is the fastest way to find the cause:

```bash
docker logs python
# Output: python: can't open file '/code/app.py': [Errno 2] No such file or directory
```

This single log line revealed exactly what was wrong — the volume mount was hiding `app.py`. Without checking `docker logs`, the root cause of the exit would not have been obvious from `docker ps -a` alone.

***

_Lab completed on 2026-03-27 | Server: stapp01 | OS: CentOS Stream 9 | Docker: 26.1.3 | Docker Compose: v5.0.2_

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
