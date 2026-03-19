# Day 19: Install and Configure Web Application
xFusionCorp Industries is planning to host two static websites on their infra in `Stratos Datacenter`. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:

a. Install `httpd` package and dependencies on `app server 1`.

b. Apache should serve on port `6300`.

c. There are two website's backups `/home/thor/blog` and `/home/thor/games` on `jump_host`. Set them up on Apache in a way that `blog` should work on the link `http://localhost:6300/blog/` and `games` should work on link `http://localhost:6300/games/` on the mentioned app server.

d. Once configured you should be able to access the website using `curl` command on the respective app server, i.e `curl http://localhost:6300/blog/` and `curl http://localhost:6300/games/`

## Host Multiple Static Websites on Apache (Port 6300)
### Objective
Prepare **App Server 1 (stapp01)** to host two static websites (`blog` and `games`) using Apache HTTPD on a custom port (**6300**).
The website backups are available on the **jump host** and must be deployed on the app server.

***

### Infrastructure Used
| Server       | Hostname                            | User   |
| ------------ | ----------------------------------- | ------ |
| Jump Host    | `jump_host.stratos.xfusioncorp.com` | `thor` |
| App Server 1 | `stapp01.stratos.xfusioncorp.com`   | `tony` |

***

### Solution Overview
* Install Apache (`httpd`) on **stapp01**
* Configure Apache to listen on **port 6300**
* Deploy two static sites:
  * `blog` → `/blog/`
  * `games` → `/games/`
* Verify accessibility using `curl`

***

### Step 1: Login to Jump Host
```bash
ssh thor@jump_host.stratos.xfusioncorp.com
```

Confirm the host authenticity and enter the password when prompted.

***

### Step 2: Copy Website Backups to App Server 1
From the **jump host**, copy both website directories to `stapp01`:

```bash
scp -r /home/thor/blog tony@stapp01.stratos.xfusioncorp.com:/tmp/
scp -r /home/thor/games tony@stapp01.stratos.xfusioncorp.com:/tmp/
```

This transfers the static website content to the app server for deployment.

***

### Step 3: Login to App Server 1
```bash
ssh tony@stapp01.stratos.xfusioncorp.com
```

***

### Step 4: Install Apache (httpd)
Install Apache and its dependencies using `yum`:

```bash
sudo yum install -y httpd
```

Ensure the installation completes successfully.

***

### Step 5: Configure Apache to Listen on Port 6300
Edit the Apache configuration file:

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

Update the `Listen` directive:

```apache
Listen 6300
```

Save and exit the file.

***

### Step 6: Deploy Website Content
Move the website directories to Apache’s document root:

```bash
sudo mkdir -p /var/www/html
sudo mv /tmp/blog /var/www/html/
sudo mv /tmp/games /var/www/html/
```

Set correct ownership and permissions:

```bash
sudo chown -R apache:apache /var/www/html/blog /var/www/html/games
sudo chmod -R 755 /var/www/html/blog /var/www/html/games
```

***

### Step 7: Start and Enable Apache Service
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

This ensures Apache starts immediately and on system boot.

***

### Step 8: Verification
Verify that both websites are accessible locally on **port 6300**:

```bash
curl http://localhost:6300/blog/
curl http://localhost:6300/games/
```

#### Expected Output
**Blog Website**

```html
<h1>KodeKloud</h1>
<p>This is a sample page for our blog website</p>
```

**Games Website**

```html
<h1>KodeKloud</h1>
<p>This is a sample page for our games website</p>
```

***

### Final Result
 Apache installed on **stapp01**
 Apache listening on **port 6300**
 Blog site accessible at `/blog/`
 Games site accessible at `/games/`
 Verified using `curl`

***

### Task Status
 **CONGRATULATIONS!**
The task was completed successfully and validated by the system.

**Reference ID:** `680774af399a2462b6cc6670`
