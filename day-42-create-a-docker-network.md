# Day 42: Create a Docker Network

The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:

a. Create a docker network named as `media` on App Server `1` in `Stratos DC`.

b. Configure it to use `bridge` drivers.

c. Set it to use subnet `192.168.0.0/24` and iprange `192.168.0.0/24`.

***

## Create Docker Network with Custom Subnet

The Nautilus DevOps team needs to set up several Docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some Docker networks to be used later.

Complete the task based on the following ticket description:

* Create a Docker network named **`media`** on **App Server 1 (`stapp01`)** in Stratos DC.
* Configure it to use the **bridge driver**.
* Set the **subnet** to **`172.168.0.0/24`**.
* Set the **IP range** to **`172.168.0.0/24`**.

***

In this task, we:

1. Connect to **App Server 1 (`stapp01`)** from the **jump host**.
2. Create a **custom Docker network**.
3. Configure it with:
   * **Bridge driver**
   * **Custom subnet**
   * **Custom IP range**
4. Verify the network creation.

Docker networks allow containers to communicate with each other in an isolated environment.

***

## Step 1: Connect to App Server 1

We are already logged into the **jump host**, so we directly SSH into **stapp01**.

```bash
ssh tony@stapp01
```

#### Terminal Output

```bash
thor@jump-host ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (10.244.81.11)' can't be established.
ED25519 key fingerprint is SHA256:lSTfj4i83ZLEhBAIbcVaPV2xSkzcAA1KAbx8k1ClWiw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
```

***

## Step 2: Create the Docker Network

Now create the Docker network using the **bridge driver** with the specified **subnet and IP range**.

```bash
docker network create \
--driver bridge \
--subnet 172.168.0.0/24 \
--ip-range 172.168.0.0/24 \
media
```

#### Terminal Output

```bash
[tony@stapp01 ~]$ docker network create --driver bridge --subnet 172.168.0.0/24 --ip-range 172.168.0.0/24 media
c982992f7fb9512db0b6134f5fb06618792801d433920ef6fb43fb3c0cfef53e
```

Docker returns a **network ID**, confirming that the network has been created successfully.

***

## Step 3: Verify the Network

List all Docker networks to confirm that **media** has been created.

```bash
docker network ls
```

#### Terminal Output

```bash
[tony@stapp01 ~]$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
76fe0573d7c7   bridge    bridge    local
25e9ce887095   host      host      local
c982992f7fb9   media     bridge    local
65bb9fd6be6f   none      null      local
```

***

## Result

The Docker network **`media`** was successfully created with:

* **Driver:** bridge
* **Subnet:** 172.168.0.0/24
* **IP Range:** 172.168.0.0/24
* **Server:** stapp01

This network can now be used by Docker containers for application deployment.



<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>



## **Level 2**

The Nautilus DevOps team needs to set up several docker environments for different applications. One of the team members has been assigned a ticket where he has been asked to create some docker networks to be used later. Complete the task based on the following ticket description:

a. Create a docker network named as `beta` on App Server `3` in `Stratos DC`.

b. Configure it to use `bridge` drivers.

c. Set it to use subnet `10.10.1.0/24` and iprange `10.10.1.0/24`.



```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.81.19)' can't be established.
ED25519 key fingerprint is SHA256:LaLUbprcvX2bAGEEV24wAUUd9onp9twCQNBCKJLNdL0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password: 
[banner@stapp03 ~]$ docker network create \
> --driver bridge \
> --subnet 10.10.1.0/24 \
> --ip-range 10.10.1.0/24 \
> beta
bcb73a00216c07c4eb5fc181f392e857f1e07f083c3ecff32e2686e37cd38fba
[banner@stapp03 ~]$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
bcb73a00216c   beta      bridge    local
46ead9a11dd0   bridge    bridge    local
7c8b613b88d2   host      host      local
048894f0ce01   none      null      local
[banner@stapp03 ~]$ docker network inspect beta
[
    {
        "Name": "beta",
        "Id": "bcb73a00216c07c4eb5fc181f392e857f1e07f083c3ecff32e2686e37cd38fba",
        "Created": "2026-03-25T02:11:45.838215938Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.10.1.0/24",
                    "IPRange": "10.10.1.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[banner@stapp03 ~]$ 
```

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
