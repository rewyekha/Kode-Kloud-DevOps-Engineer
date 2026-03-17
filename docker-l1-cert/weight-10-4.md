# Weight: 10

The Nautilus DevOps team is planning to setup/create some docker containers on `App Server 2` in `Stratos Datacenter`, some prerequisites are needs to be done on this server. Find below more details:

Create a new network named `mysql-network` using the `bridge` driver. Allocate subnet `182.18.0.0/24`, configure Gateway `182.18.0.1`.

**Run this command on App Server 2**

```bash
sudo docker network create \
--driver bridge \
--subnet 182.18.0.0/24 \
--gateway 182.18.0.1 \
mysql-network
```

**Explanation**

* `network create` → Creates a new Docker network
* `--driver bridge` → Uses the bridge network driver
* `--subnet 182.18.0.0/24` → Defines the subnet range
* `--gateway 182.18.0.1` → Sets the gateway for the network
* `mysql-network` → Name of the network

***

**Verify the network**

```bash
sudo docker network ls
```

You should see:

```
NETWORK ID     NAME            DRIVER    SCOPE
xxxxx          mysql-network   bridge    local
```

For detailed confirmation:

```bash
sudo docker network inspect mysql-network
```

You should see:

* Subnet → **182.18.0.0/24**
* Gateway → **182.18.0.1**

✅ Network setup complete.

Send **8/9** when ready. 🚀

```bash
[steve@stapp02 ~]$ sudo docker network create \
--driver bridge \
--subnet 182.18.0.0/24 \
--gateway 182.18.0.1 \
mysql-network
aa4e9fee72960b9f39b6a10c504f09363af43e1b8de0b300da2fe60ab688ac78
[steve@stapp02 ~]$ sudo docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
168d0f22eeac   bridge          bridge    local
721e973e66a7   host            host      local
aa4e9fee7296   mysql-network   bridge    local
fd5e3fdd9b67   none            null      local
[steve@stapp02 ~]$ sudo docker network inspect mysql-network
[
    {
        "Name": "mysql-network",
        "Id": "aa4e9fee72960b9f39b6a10c504f09363af43e1b8de0b300da2fe60ab688ac78",
        "Created": "2026-03-12T08:45:44.101083343Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "182.18.0.0/24",
                    "Gateway": "182.18.0.1"
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
[steve@stapp02 ~]$ 
```
