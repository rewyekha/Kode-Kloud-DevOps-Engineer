# Untitled

The production support team of `xFusionCorp Industries` is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for taking websites backup. They have a static website running on `App Server 3` in `Stratos Datacenter`, and they need to create a bash script named `official_backup.sh` which should accomplish the following tasks. (Also remember to place the script under `/scripts` directory on `App Server 3`).\
\
a. Create a zip archive named `xfusioncorp_official.zip` of `/var/www/html/official` directory.

b. Save the archive in `/backup/` on `App Server 3`. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on `Nautilus Backup Server`.\
\
c. Copy the created archive to `Nautilus Backup Server` server in `/backup/` location.

d. Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, `tony` in case of `App Server 1`) must be able to run it.

e. Do not use sudo inside the script.

Note:\
The zip package must be installed on given App Server before executing the script. This package is essential for creating the zip archive of the website files. Install it manually outside the script.



### What you need to do (summary)

* Work **on App Server 3** → `stapp03` (user: `banner`)
* Create a script:\
  &#xNAN;**`/scripts/official_backup.sh`**
* Script must:
  1. Zip `/var/www/html/official`
  2. Save zip as `xfusioncorp_official.zip` in `/backup`
  3. Copy the zip to **Nautilus Backup Server** (`stbkp01`) → `/backup`
  4. Copy must be **passwordless**
  5. **No sudo** inside the script

***

### Step 1: Login to App Server 3

```bash
ssh banner@stapp03
```

***

### Step 2: Install zip (outside the script)

```bash
sudo yum install -y zip
```

(Allowed because install is **outside** the script)

***

### Step 3: Setup passwordless SSH to Backup Server

This is **mandatory**, otherwise the script will prompt for a password.

#### Generate SSH key (if not already present)

```bash
ssh-keygen -t rsa
```

Press **Enter** for all prompts (no passphrase).

#### Copy key to Backup Server

```bash
ssh-copy-id clint@stbkp01
```

Password: `H@wk3y3`

Verify:

```bash
ssh clint@stbkp01
```

If it logs in **without password**, you’re good.

Exit:

```bash
exit
```

***

### Step 4: Create required directories (if not present)

```bash
mkdir -p /scripts
mkdir -p /backup
```

***

### Step 5: Create the script

```bash
vi /scripts/official_backup.sh
```

#### **Script content (VERY IMPORTANT)**

```bash
#!/bin/bash

# Variables
BACKUP_NAME="xfusioncorp_official.zip"
SOURCE_DIR="/var/www/html/official"
LOCAL_BACKUP_DIR="/backup"
REMOTE_USER="clint"
REMOTE_HOST="stbkp01"
REMOTE_BACKUP_DIR="/backup"

# Create zip archive
zip -r ${LOCAL_BACKUP_DIR}/${BACKUP_NAME} ${SOURCE_DIR}

# Copy backup to Nautilus Backup Server
scp ${LOCAL_BACKUP_DIR}/${BACKUP_NAME} ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_BACKUP_DIR}/
```

Save and exit (`:wq`).

***

### Step 6: Make script executable

```bash
chmod +x /scripts/official_backup.sh
```

***

### Step 7: Test the script

```bash
/scripts/official_backup.sh
```

✔ No password prompt\
✔ Zip created in `/backup`\
✔ Zip copied to `stbkp01:/backup`

***

### Step 8: Verify on Backup Server (optional but recommended)

```bash
ssh clint@stbkp01
ls /backup
```

You should see:

```
xfusioncorp_official.zip
```

***

### ✅ Final Checklist (Exam-safe)

* ✔ Script name: `official_backup.sh`
* ✔ Location: `/scripts`
* ✔ Zip name: `xfusioncorp_official.zip`
* ✔ No `sudo` in script
* ✔ Passwordless SSH
* ✔ Uses `zip`
* ✔ Copies to Nautilus Backup Server

<pre><code><strong>thor@jumphost ~$ ssh banner@stapp03
</strong>
The authenticity of host 'stapp03 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:65rCjQIvUgA6KKsa5Iu2Yz4YykLrH7haJkQ7xKj6+Go.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password: 
[banner@stapp03 ~]$ sudo yum install -y zip

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
CentOS Stream 9 - BaseOS                                                                    31 kB/s | 5.9 kB     00:00    
CentOS Stream 9 - BaseOS                                                                    14 MB/s | 8.8 MB     00:00    
CentOS Stream 9 - AppStream                                                                 21 kB/s | 6.3 kB     00:00    
CentOS Stream 9 - AppStream                                                                 14 MB/s |  26 MB     00:01    
CentOS Stream 9 - Extras packages                                                           33 kB/s | 8.0 kB     00:00    
CentOS Stream 9 - Extras packages                                                           20 kB/s |  20 kB     00:00    
Docker CE Stable - x86_64                                                                   24 kB/s | 2.0 kB     00:00    
Docker CE Stable - x86_64                                                                  249 kB/s |  65 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64                                              37 kB/s | 9.0 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64                                             9.2 MB/s |  20 MB     00:02    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                       3.2 kB/s | 993  B     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                                       88 kB/s |  22 kB     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                                      480 kB/s | 259 kB     00:00    
Dependencies resolved.
===========================================================================================================================
 Package                    Architecture                Version                          Repository                   Size
===========================================================================================================================
Installing:
 zip                        x86_64                      3.0-35.el9                       baseos                      266 k
Installing dependencies:
 unzip                      x86_64                      6.0-59.el9                       baseos                      182 k

Transaction Summary
===========================================================================================================================
Install  2 Packages

Total download size: 447 k
Installed size: 1.1 M
Downloading Packages:
(1/2): unzip-6.0-59.el9.x86_64.rpm                                                         854 kB/s | 182 kB     00:00    
(2/2): zip-3.0-35.el9.x86_64.rpm                                                           1.0 MB/s | 266 kB     00:00    
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                      1.0 MB/s | 447 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                   1/1 
  Installing       : unzip-6.0-59.el9.x86_64                                                                           1/2 
  Installing       : zip-3.0-35.el9.x86_64                                                                             2/2 
  Running scriptlet: zip-3.0-35.el9.x86_64                                                                             2/2 
  Verifying        : unzip-6.0-59.el9.x86_64                                                                           1/2 
  Verifying        : zip-3.0-35.el9.x86_64                                                                             2/2 

Installed:
  unzip-6.0-59.el9.x86_64                                       zip-3.0-35.el9.x86_64                                      

Complete!
<strong>[banner@stapp03 ~]$ ssh-keygen -t rsa
</strong>Generating public/private rsa key pair.
Enter file in which to save the key (/home/banner/.ssh/id_rsa): 
Created directory '/home/banner/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/banner/.ssh/id_rsa
Your public key has been saved in /home/banner/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:MjVxGACby70yvCvrSYQ931QCWrDvbANXwssrWRp5ia4 banner@stapp03.stratos.xfusioncorp.com
The key's randomart image is:
+---[RSA 3072]----+
|  ..+...oo.      |
|   = +  .o       |
|  o = o +        |
| o * B + .       |
|. O @ = S        |
| o # + +         |
|  * @ o          |
| o.+ =           |
|E.+oo.           |
+----[SHA256]-----+
<strong>[banner@stapp03 ~]$ ssh-copy-id clint@stbkp01
</strong>/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/banner/.ssh/id_rsa.pub"
The authenticity of host 'stbkp01 (172.16.238.16)' can't be established.
ED25519 key fingerprint is SHA256:SBeLdYjJtmSBXg+mvU7WodjNZItCEqCsCTIDVRPTYwU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
clint@stbkp01's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'clint@stbkp01'"
and check to make sure that only the key(s) you wanted were added.

<strong>[banner@stapp03 ~]$ ssh clint@stbkp01
</strong><strong>[clint@stbkp01 ~]$ exit
</strong>logout
Connection to stbkp01 closed.
<strong>[banner@stapp03 ~]$ ls
</strong><strong>[banner@stapp03 ~]$ mkdir -p /scripts
</strong>mkdir -p /backup
<strong>[banner@stapp03 ~]$ ls
</strong><strong>[banner@stapp03 ~]$ vi /scripts/official_backup.sh
</strong><strong>[banner@stapp03 ~]$ chmod +x /scripts/official_backup.sh
</strong><strong>[banner@stapp03 ~]$ /scripts/official_backup.sh
</strong>  adding: var/www/html/official/ (stored 0%)
  adding: var/www/html/official/.gitkeep (stored 0%)
  adding: var/www/html/official/index.html (stored 0%)
xfusioncorp_official.zip                                                                 100%  616     2.2MB/s   00:00    
<strong>[banner@stapp03 ~]$ ssh clint@stbkp01
</strong>ls /backup
Last login: Thu Dec 25 04:33:24 2025 from 172.16.238.12
<strong>[clint@stbkp01 ~]$ ls /backup
</strong>xfusioncorp_official.zip
[clint@stbkp01 ~]$ 
</code></pre>
