# Day 15: Setup SSL for Nginx

The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 1 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below: 1. Install and configure nginx on App Server 1. 2. On App Server 1 there is a self signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx. 3. Create an index.html file with content Welcome! under Nginx document root. 4. For final testing try to access the App Server 1 link (either hostname or IP) from jump host using curl command. For example curl -Ik https:///.

#### The Task

The system admins team of **xFusionCorp Industries** needs to deploy a new application on **App Server 1** in **Stratos Datacenter**. The server must be prepared with Nginx and SSL before deployment.

#### Requirements

1. Install and configure **Nginx** on **App Server 1**
2. Deploy the provided **self-signed SSL certificate**
3. Create an **index.html** page with content `Welcome!`
4. Verify HTTPS access from the **jump host** using `curl`

***

### Commands I Ran — One-liners + What Each Does

***

#### 1. Login to App Server 1

```bash
ssh banner@stapp01
```

Connects to App Server 1 using the `banner` user.

***

#### 2. Install Nginx

```bash
sudo yum install nginx -y
```

Installs the Nginx web server.\
`-y` automatically confirms the installation.

***

#### 3. Move SSL Certificate and Key

```bash
sudo mv /tmp/nautilus.crt /etc/pki/tls/certs/
sudo mv /tmp/nautilus.key /etc/pki/tls/private/
```

Moves the self-signed SSL certificate and key to standard TLS directories used by Nginx.

***

#### 4. Configure Nginx for SSL

Edit the main configuration file:

```bash
sudo vi /etc/nginx/nginx.conf
```

Add or update the HTTPS server block:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name <APP_SERVER_1_IP>;

    root /usr/share/nginx/html;
    index index.html;

    ssl_certificate "/etc/pki/tls/certs/nautilus.crt";
    ssl_certificate_key "/etc/pki/tls/private/nautilus.key";

    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;
}
```

🔹 Replace `<APP_SERVER_1_IP>` with the actual IP or hostname of App Server 1.

***

#### 5. Test Nginx Configuration

```bash
sudo nginx -t
```

Validates the Nginx configuration syntax.

***

#### 6. Enable and Start Nginx

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

Starts Nginx immediately and enables it at boot.

***

#### 7. Create Welcome Page

```bash
sudo vi /usr/share/nginx/html/index.html
```

Add the following content:

```html
Welcome!
```

This is the default page served over HTTPS.

***

#### 8. Verify from Jump Host

From the jump host, run:

```bash
curl -Ik https://<APP_SERVER_1_IP>/
```

Expected result:

* `HTTP/1.1 200 OK`
* SSL certificate details in headers

***

### Summary: What I Learned

* Installing and enabling **Nginx** on a CentOS-based system
* Deploying **self-signed SSL certificates** correctly
* Configuring **HTTPS (TLS)** in Nginx
* Creating and serving static content
* Verifying secure connectivity using `curl`

***

✅ **App Server 1 is now fully prepared for secure application deployment.**

*
* Optimize SSL settings
* Automate the setup using **Ansible**
