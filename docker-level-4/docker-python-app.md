# Docker Python App

A python app needed to be Dockerized, and then it needs to be deployed on `App Server 2`. We have already copied a `requirements.txt` file (having the app dependencies) under `/python_app/src/` directory on `App Server 2`. Further complete this task as per details mentioned below:\
<br>

1. Create a `Dockerfile` under `/python_app` directory:
   * Use any `python` image as the base image.
   * Install the dependencies using `requirements.txt` file.
   * Expose the port `3000`.
   * Run the `server.py` script using `CMD`.\
     <br>
2. Build an image named `nautilus/python-app` using this Dockerfile.\
   <br>
3. Once image is built, create a container named `pythonapp_nautilus`:
   * Map port `3000` of the container to the host port `8094`.\
     <br>
4. Once deployed, you can test the app using `curl` command on `App Server 2`.\
   <br>

```sh
curl http://localhost:8094/
```

```bash
thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.49.22)' can't be established.
ED25519 key fingerprint is SHA256:Vk8ZTRHQz8hsTx9xLrUuwtTytB2sPYfnGrgIob9Hyp4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
Permission denied, please try again.
steve@stapp02's password: 
Last failed login: Tue Mar 31 12:26:34 UTC 2026 from 10.244.30.61 on ssh:notty
There was 1 failed login attempt since the last successful login.
[steve@stapp02 ~]$ ls
[steve@stapp02 ~]$ cd /python_app/
[steve@stapp02 python_app]$ ls
src
[steve@stapp02 python_app]$ cd /src/
-bash: cd: /src/: No such file or directory
[steve@stapp02 python_app]$ cd /src
-bash: cd: /src: No such file or directory
[steve@stapp02 python_app]$ cat /src/requirements.txt
cat: /src/requirements.txt: No such file or directory
[steve@stapp02 python_app]$ cd /python_app/src/
[steve@stapp02 src]$ ls
requirements.txt  server.py
[steve@stapp02 src]$ cat server.py
from flask import Flask

# the all-important app variable:
app = Flask(__name__)

@app.route("/")
def hello():
    return "Welcome to xFusionCorp Industries!"

if __name__ == "__main__":
        app.config['TEMPLATES_AUTO_RELOAD'] = True
        app.run(host='0.0.0.0', debug=True, port=3000)[steve@stapp02 src]$ 
[steve@stapp02 src]$ cat requirements.txt
flask[steve@stapp02 src]$ cd ..
[steve@stapp02 python_app]$ sudo vi Dockerfile

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 python_app]$ cat Dockerfile
FROM flask:lastest
WORKDIR /python_app
COPY server.py
RUN py install flask
EXPOSE 3000
CMD ["flask"]
[steve@stapp02 python_app]$ sudo vi Dockerfile
[steve@stapp02 python_app]$ cat Dockerfile
FROM python:3.9

WORKDIR /python_app

COPY src/ /python_app/

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 3000

CMD ["python", "server.py"]
[steve@stapp02 python_app]$ cat /python_app/src/server.js
cat: /python_app/src/server.js: No such file or directory
[steve@stapp02 python_app]$ cat /python_app/src/server.py
from flask import Flask

# the all-important app variable:
app = Flask(__name__)

@app.route("/")
def hello():
    return "Welcome to xFusionCorp Industries!"

if __name__ == "__main__":
        app.config['TEMPLATES_AUTO_RELOAD'] = True
        app.run(host='0.0.0.0', debug=True, port=3000)[steve@stapp02 python_app]$ 
[steve@stapp02 python_app]$ cat /python_app/src/requirements.txt
flask[steve@stapp02 python_app]$ 
[steve@stapp02 python_app]$ docker build -t nautilus/python-app /python_app
[+] Building 30.8s (9/9) FINISHED                                                                               docker:default
 => [internal] load build definition from Dockerfile                                                                      0.1s
 => => transferring dockerfile: 194B                                                                                      0.0s
 => [internal] load metadata for docker.io/library/python:3.9                                                             1.6s
 => [internal] load .dockerignore                                                                                         0.1s
 => => transferring context: 2B                                                                                           0.0s
 => [1/4] FROM docker.io/library/python:3.9@sha256:da5aee29682d12a6649f51c8d6f15b87deb3e6c524b923c41d0cb3304d07c913      21.0s
 => => resolve docker.io/library/python:3.9@sha256:da5aee29682d12a6649f51c8d6f15b87deb3e6c524b923c41d0cb3304d07c913       0.1s
 => => sha256:d6ca7d9522a172c424721d3509ee12079f7864a742b6adf1eeb66b6c405307ee 2.32kB / 2.32kB                            0.0s
 => => sha256:da5aee29682d12a6649f51c8d6f15b87deb3e6c524b923c41d0cb3304d07c913 10.30kB / 10.30kB                          0.0s
 => => sha256:bcd3da5974912584a81ed86fd944ab5fba9093ff1c9a0b0ed18349f9a69e4762 6.23kB / 6.23kB                            0.0s
 => => sha256:89d573bf42b377ce6a5a0451c15388849686fa4058efd68999f3b014daeb5b55 25.62MB / 25.62MB                          0.9s
 => => sha256:26dfe2fac1c486e9aaf41d1028ed30be2c442aa84af44462bc7bac8c148ffb13 67.78MB / 67.78MB                          2.4s
 => => sha256:795dbedde24d2c72dafd2b71fe36643552e56859c0e29cdb095ed54b825fbaa2 49.28MB / 49.28MB                          1.8s
 => => sha256:79d5bd8a8d262418bf22e705535ce38c6789dc72e319d76b30aafa5c331b6924 235.93MB / 235.93MB                        5.8s
 => => extracting sha256:795dbedde24d2c72dafd2b71fe36643552e56859c0e29cdb095ed54b825fbaa2                                 1.8s
 => => sha256:081ccf923272c30c6072c6ff1617d9072e03ab2a90a431951d325d45e296962b 6.10MB / 6.10MB                            2.3s
 => => sha256:c9723aa529b03c40e66d0aee927a410b4719528ab865af6e0bac1b7c9b10829e 20.37MB / 20.37MB                          3.1s
 => => sha256:91c91c91f1d23f4edf4280a8fe935f14340fec43a7a3576149a7cffcf70c2f9b 250B / 250B                                2.8s
 => => extracting sha256:89d573bf42b377ce6a5a0451c15388849686fa4058efd68999f3b014daeb5b55                                 2.1s
 => => extracting sha256:26dfe2fac1c486e9aaf41d1028ed30be2c442aa84af44462bc7bac8c148ffb13                                 3.0s
 => => extracting sha256:79d5bd8a8d262418bf22e705535ce38c6789dc72e319d76b30aafa5c331b6924                                 8.7s
 => => extracting sha256:081ccf923272c30c6072c6ff1617d9072e03ab2a90a431951d325d45e296962b                                 0.3s
 => => extracting sha256:c9723aa529b03c40e66d0aee927a410b4719528ab865af6e0bac1b7c9b10829e                                 0.8s
 => => extracting sha256:91c91c91f1d23f4edf4280a8fe935f14340fec43a7a3576149a7cffcf70c2f9b                                 0.1s
 => [internal] load build context                                                                                         0.1s
 => => transferring context: 401B                                                                                         0.0s
 => [2/4] WORKDIR /python_app                                                                                             0.2s
 => [3/4] COPY src/ /python_app/                                                                                          0.1s
 => [4/4] RUN pip install --no-cache-dir -r requirements.txt                                                              3.8s
 => exporting to image                                                                                                    3.6s 
 => => exporting layers                                                                                                   3.5s 
 => => writing image sha256:7502bf6fb6429e72516b61ff890427006a73a0300dd331d1969e748ed355612e                              0.0s 
 => => naming to docker.io/nautilus/python-app                                                                            0.0s 
[steve@stapp02 python_app]$ docker run -d -p 8094:3000 --name pythonapp_nautilus nautilus/python-app                           
e96baf43bb2a997948ae2c2847ee036f644ec1c3f8928f6c49a1cbd67125aa47                                                               
[steve@stapp02 python_app]$ curl http://localhost:8094/
Welcome to xFusionCorp Industries![steve@stapp02 python_app]$ thor@jump-host ~$ ssh steve@stapp02
The authenticity of host 'stapp02 (10.244.49.22)' can't be established.
ED25519 key fingerprint is SHA256:Vk8ZTRHQz8hsTx9xLrUuwtTytB2sPYfnGrgIob9Hyp4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02' (ED25519) to the list of known hosts.
steve@stapp02's password: 
Permission denied, please try again.
steve@stapp02's password: 
Last failed login: Tue Mar 31 12:26:34 UTC 2026 from 10.244.30.61 on ssh:notty
There was 1 failed login attempt since the last successful login.
[steve@stapp02 ~]$ ls
[steve@stapp02 ~]$ cd /python_app/
[steve@stapp02 python_app]$ ls
src
[steve@stapp02 python_app]$ cd /src/
-bash: cd: /src/: No such file or directory
[steve@stapp02 python_app]$ cd /src
-bash: cd: /src: No such file or directory
[steve@stapp02 python_app]$ cat /src/requirements.txt
cat: /src/requirements.txt: No such file or directory
[steve@stapp02 python_app]$ cd /python_app/src/
[steve@stapp02 src]$ ls
requirements.txt  server.py
[steve@stapp02 src]$ cat server.py
from flask import Flask

# the all-important app variable:
app = Flask(__name__)

@app.route("/")
def hello():
    return "Welcome to xFusionCorp Industries!"

if __name__ == "__main__":
        app.config['TEMPLATES_AUTO_RELOAD'] = True
        app.run(host='0.0.0.0', debug=True, port=3000)[steve@stapp02 src]$ 
[steve@stapp02 src]$ cat requirements.txt
flask[steve@stapp02 src]$ cd ..
[steve@stapp02 python_app]$ sudo vi Dockerfile

We trust you have received the usual lecture from the local System
Welcome to xFusionCorp Industries![steve@stapp02 python_app]$ ^C                                                               
[steve@stapp02 python_app]$ 
```



<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
