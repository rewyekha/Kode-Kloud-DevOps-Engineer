# Day 16: Install and Configure Nginx as an LBR

Absolutely! Here's a clean **GitBook-style documentation** for your Nginx LBR lab, including the problem statement and step-by-step solution:

***

## Day 16: Install and Configure Nginx as a Load Balancer (LBR)

### Objective

The Nautilus production support team has observed increasing website traffic, resulting in degraded performance. The team has decided to deploy the website on a **high availability stack** in Stratos DC. All app servers are running, and the only pending task is to configure the **Load Balancer (LBR)** using Nginx.

You are required to:

1. Install Nginx on the LBR server (`stlb01`).
2. Configure HTTP load-balancing using all three App Servers (`stapp01`, `stapp02`, `stapp03`).
3. Ensure you update **only** the main Nginx configuration file: `/etc/nginx/nginx.conf`.
4. Do not change the Apache ports already configured on the app servers.
5. Verify the website is accessible via the **StaticApp** button.

***

### Infrastructure Details

| Server Name | IP            | Hostname                        | User   | Password | Purpose           |
| ----------- | ------------- | ------------------------------- | ------ | -------- | ----------------- |
| stapp01     | 172.16.238.10 | stapp01.stratos.xfusioncorp.com | tony   | Ir0nM@n  | Nautilus App 1    |
| stapp02     | 172.16.238.11 | stapp02.stratos.xfusioncorp.com | steve  | Am3ric@  | Nautilus App 2    |
| stapp03     | 172.16.238.12 | stapp03.stratos.xfusioncorp.com | banner | BigGr33n | Nautilus App 3    |
| stlb01      | 172.16.238.14 | stlb01.stratos.xfusioncorp.com  | loki   | Mischi3f | Nautilus HTTP LBR |

> Note: Apache service on app servers may run on a non-standard port (in this lab, it is **5001**).

***

### Step 1: Install Nginx on LBR

SSH into the load balancer server:

```bash
thor@jumphost ~$ ssh loki@stlb01
```

Install and enable Nginx:

```bash
sudo yum install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

***

### Step 2: Verify Apache Port on App Servers

SSH into each app server and check the Apache port:

```bash
sudo ss -tulpn | grep httpd
```

Example output on `stapp01`:

```
tcp   LISTEN 0      511          0.0.0.0:5001       0.0.0.0:*    users:(("httpd",pid=1670,fd=3),("httpd",pid=1669,fd=3),("httpd",pid=1668,fd=3),("httpd",pid=1660,fd=3))
```

> Apache is running on **port 5001**. Make a note of this for all app servers.

***

### Step 3: Configure Nginx for Load Balancing

Edit the main Nginx configuration file:

```bash
sudo vi /etc/nginx/nginx.conf
```

Add the upstream block and proxy configuration **inside the `http { ... }` block**:

```nginx
http {
    # existing config ...

    upstream nautilus_apps {
        server 172.16.238.10:5001;
        server 172.16.238.11:5001;
        server 172.16.238.12:5001;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://nautilus_apps;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

***

### Step 4: Test Nginx Configuration

```bash
sudo nginx -t
```

You should see:

```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

***

### Step 5: Verify Load Balancing

* Open the **StaticApp** button in the lab interface.
* Refresh multiple times; traffic should round-robin across all three app servers.

***

### Troubleshooting

* **502 Bad Gateway**: Usually occurs if Nginx is forwarding to the wrong port. Ensure the `upstream` block uses the correct Apache ports (5001 in this lab).
* `nginx -t` syntax errors: Check for typos, especially in directives like `proxy_set_header` (it must have underscores, not hyphens).

***

### ✅ Result

* Nginx installed on LBR.
* Load balancing configured correctly.
* Website is accessible and traffic is evenly distributed to all app servers.

***

This GitBook doc can be saved under something like:

```
labs/16-nginx-lbr/README.md
```

***

If you want, I can also write a **ready-to-use full `/etc/nginx/nginx.conf` file** for this lab that you can directly copy-paste. It will include all default configs plus the upstream block.

Do you want me to do that?
