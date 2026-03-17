# Weight: 10

The Nautilus DevOps team is planning to do some cleanup on `App Server 2` in `Stratos Datacenter`, some old and unused docker networks need to be deleted. Find below more details:

Delete a docker network named `php-network` from `App Server 2` in `Stratos Datacenter`.

To complete this task, simply **remove the Docker network `php-network`** from **App Server 2**.

**1. Remove the network**

```bash
sudo docker network rm php-network
```

***

**2. Verify the network is deleted**

```bash
sudo docker network ls
```

Ensure **`php-network`** no longer appears in the list.

✅ This will complete task **8/9**.

Send **9/9** when you're ready. 🚀

```bash
[steve@stapp02 ~]$ sudo docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
168d0f22eeac   bridge          bridge    local
721e973e66a7   host            host      local
aa4e9fee7296   mysql-network   bridge    local
fd5e3fdd9b67   none            null      local
0cf6dea21112   php-network     bridge    local
[steve@stapp02 ~]$ sudo docker network rm php-network
php-network
[steve@stapp02 ~]$ sudo docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
168d0f22eeac   bridge          bridge    local
721e973e66a7   host            host      local
aa4e9fee7296   mysql-network   bridge    local
fd5e3fdd9b67   none            null      local
[steve@stapp02 ~]$ 
```
