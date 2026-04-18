# Day 20: Configure Nginx + PHP-FPM Using Unix Sock

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The `Nautilus` application development team is planning to launch a new PHP-based application, which they want to deploy on `Nautilus` infra in `Stratos DC`. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:

a. Install `nginx` on `app server 3` , configure it to use port `8099` and its document root should be `/var/www/html`.

b. Install `php-fpm` version `8.3` on `app server 3`, it must use the unix socket `/var/run/php-fpm/default.sock` (create the parent directories if don't exist).

c. Configure php-fpm and nginx to work together.

d. Once configured correctly, you can test the website using `curl http://stapp03:8099/index.php` command from jump host.

NOTE: We have copied two files, `index.php` and `info.php`, under `/var/www/html` as part of the `PHP-based application` setup. Please do not modify these files.


## Configure Nginx & PHP-FPM 8.3 Using Unix Socket on App Server 3

### Objective

Deploy a PHP-based application on **stapp03** using **nginx** and **PHP-FPM 8.3**, configured to communicate via a Unix socket and validated using `curl` from the jump host.

***

### Requirements

* Nginx must listen on **port 8099**
* Document root: `/var/www/html`
* PHP-FPM version: **8.3**
* PHP-FPM socket: `/var/run/php-fpm/default.sock`
* Do **not** modify `index.php` or `info.php`
* Validate using:

    ```bash
    curl http://stapp03:8099/index.php
    ```

***

### Step 1: Install and Configure Nginx

#### Install nginx

```bash
sudo yum install -y nginx
```

#### Create nginx configuration for port 8099

```bash
sudo vi /etc/nginx/conf.d/php_app_8099.conf
```

Add the following configuration:

```nginx
server {
    listen 8099;
    server_name stapp03;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### Validate and restart nginx

```bash
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl restart nginx
```

***

### Step 2: Install PHP-FPM 8.3

#### Enable Remi repository and PHP 8.3 module

```bash
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.3 -y
```

#### Install PHP-FPM

```bash
sudo dnf install -y php-fpm php-cli
```

#### Verify PHP version

```bash
php -v
```

Expected output: **PHP 8.3.x**

***

### Step 3: Configure PHP-FPM to Use Unix Socket

#### Edit PHP-FPM pool configuration

```bash
sudo vi /etc/php-fpm.d/www.conf
```

Update the following values:

```ini
user = nginx
group = nginx

listen = /var/run/php-fpm/default.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

#### Create socket directory

```bash
sudo mkdir -p /var/run/php-fpm
```

#### Restart PHP-FPM

```bash
sudo systemctl enable php-fpm
sudo systemctl restart php-fpm
```

#### Verify socket creation

```bash
ls -l /var/run/php-fpm/default.sock
```

Expected output:

```
srw-rw----+ 1 root root ... /var/run/php-fpm/default.sock
```

> Note: In this environment, the socket may appear owned by `root:root` with ACLs (`+`). This is acceptable and PHP-FPM works correctly.

***

### Step 4: Validation

#### Test locally on app server

```bash
curl http://localhost:8099/index.php
```

#### Test from jump host (required)

```bash
curl http://stapp03:8099/index.php
```

Expected output:

```
Welcome to xFusionCorp Industries!
```

***

### Final Result

* Nginx running on port **8099**
* PHP-FPM **8.3** configured with Unix socket
* Nginx and PHP-FPM integrated successfully
* Application accessible via curl
* All requirements satisfied without modifying application files

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
