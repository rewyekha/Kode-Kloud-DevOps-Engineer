# Day 47: Docker Python App
A python app needed to be Dockerized, and then it needs to be deployed on `App Server 2`. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on `App Server 2`. Further complete this task as per details mentioned below:

1. Create a `Dockerfile` under `/python_app` directory:
   * Use any `python` image as the base image.
   * Install the dependencies using `requirements.txt` file.
   * Expose the port `8087`.
   * Run the `server.py` script using `CMD`.
2. Build an image named `nautilus/python-app` using this Dockerfile.
3. Once image is built, create a container named `pythonapp_nautilus`:
   * Map port `8087` of the container to the host port `8095`.
4. Once deployed, you can test the app using `curl` command on `App Server 2`.

```sh
curl http://localhost:8095/
```

### Task
A Python application needs to be **Dockerized** and deployed on **App Server 2**.

A `requirements.txt` file containing dependencies is already available under:

```
/python_app/src/
```

#### Requirements
1. Create a **Dockerfile** under `/python_app`.
2. Use any **Python base image**.
3. Install dependencies from `requirements.txt`.
4. Expose port **8087**.
5. Run `server.py` using **CMD**.
6. Build a Docker image named:

```
nautilus/python-app
```

7. Create a container named:

```
pythonapp_nautilus
```

8. Map container port **8087 → host port 8095**.
9. Test the application using:

```bash
curl http://localhost:8095/
```

***

## Step 1: SSH into App Server 2
```bash
thor@jump-host ~$ ssh steve@stapp02
```

#### First-time SSH Warning
```bash
The authenticity of host 'stapp02 (10.244.73.171)' can't be established.
ED25519 key fingerprint is SHA256:el1B5RuWofirl2dy5YLBVIbOenCdqfF4u3Vu4/Ju8Hw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Accept the connection:

```bash
yes
```

Output:

```bash
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
```

Login:

```bash
steve@stapp02's password:
```

***

## Step 2: Verify Application Directory
```bash
ls
```

Output:

```bash
```

Navigate to application directory:

```bash
cd /python_app
```

Check files:

```bash
ls
```

Output:

```
src
```

Check recursively:

```bash
ls -R
```

Output:

```
.:
src

./src:
requirements.txt  server.py
```

Check permissions:

```bash
ls -l
```

Output:

```
total 4
drwxr-xr-x 2 root root 4096 Mar 12 03:34 src
```

Observation:

* Directory owned by **root**
* Normal user **steve** may face permission issues.

***

## Step 3: Create Dockerfile
Attempt to create Dockerfile:

```bash
vi /python_app/Dockerfile
```

Because of permissions, the file cannot be saved.

Solution: use **sudo**

```bash
sudo vi /python_app/Dockerfile
```

System warning message:

```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

#1) Respect the privacy of others.
#2) Think before you type.
#3) With great power comes great responsibility.
```

Enter password:

```
[sudo] password for steve:
```

***

### Dockerfile Content
```dockerfile
FROM python:3.9

WORKDIR /app

COPY src/ /app/

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8087

CMD ["python", "server.py"]
```

Save and exit.

***

## Step 4: Build Docker Image
Run the build command:

```bash
docker build -t nautilus/python-app /python_app
```

Output (trimmed):

```bash
[+] Building 27.0s (9/9) FINISHED

=> [internal] load build definition from Dockerfile
=> [internal] load metadata for docker.io/library/python:3.9
=> [1/4] FROM docker.io/library/python:3.9
=> [2/4] WORKDIR /app
=> [3/4] COPY src/ /app/
=> [4/4] RUN pip install --no-cache-dir -r requirements.txt
=> exporting to image
=> naming to docker.io/nautilus/python-app
```

Image successfully built.

***

## Step 5: Run Container
First attempt:

```bash
docker run -d --name pythonapp_nautilus -p 8095:8087 --nautilus/python-app
```

#### Error
```
unknown flag: --nautilus/python-app
See 'docker run --help'.
```

#### Cause
Extra `--` before image name.

***

### Correct Command
```bash
docker run -d \
--name pythonapp_nautilus \
-p 8095:8087 \
nautilus/python-app
```

Output:

```
85def4befad07344059cfe4c7f433c7e65bbf84e8cee6ad4e19cb3aaa5244fa6
```

Container started successfully.

***

## Step 6: Verify Container
```bash
docker ps
```

Output:

```
CONTAINER ID   IMAGE                 COMMAND              CREATED          STATUS          PORTS                                       NAMES
85def4befad0   nautilus/python-app   "python server.py"   15 seconds ago   Up 14 seconds   0.0.0.0:8095->8087/tcp, :::8095->8087/tcp   pythonapp_nautilus
```

Container is running and port mapping is correct.

***

## Step 7: Test the Application
```bash
curl http://localhost:8095/
```

Output:

```
Welcome to xFusionCorp Industries!
```

Application is working successfully.

***

## Final Verification Checklist
Dockerfile created in `/python_app`
Base image: `python:3.9`
Dependencies installed via `requirements.txt`
Port **8087 exposed**
Image built: `nautilus/python-app`
Container created: `pythonapp_nautilus`
Port mapped **8095 → 8087**
Application accessible via curl

***

**Task Completed Successfully**
