# Day 73: Jenkins Scheduled Jobs

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The devops team of xFusionCorp Industries is working on to setup centralised logging management system to maintain and analyse server logs easily. Since it will take some time to implement, they wanted to gather some server logs on a regular basis. At least one of the app servers is having issues with the Apache server. The team needs Apache logs so that they can identify and troubleshoot the issues easily if they arise. So they decided to create a Jenkins job to collect logs from the server. Please create/configure a Jenkins job as per details mentioned below:

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`\
**1.** Create a Jenkins jobs named `copy-logs`.\
**2.** Configure it to periodically build `every 3 minutes` to copy the Apache logs (both `access_log` and `error_log`) from **App Server 1** (stapp01) from the default logs location to location `/usr/src/data` on the Storage Server.

**3.** Build the job at least once so that the logs are copied and can be verified.

`Note:`\
**1.** You might need to install some plugins and restart Jenkins. We recommend selecting `Restart Jenkins when installation is complete and no jobs are running` in the update centre. Refresh the page if the UI gets stuck after a restart.\
**2.** Define the cron expression as required (e.g. `*/10 * * * *` to run every 10 minutes).\
**3.** For scenarios that require web UI changes, take screenshots or record your work (e.g. using loom.com) so you can share it for review if the task is marked incomplete.


## Jenkins Job: Centralized Apache Log Collection (copy-logs)

### Overview


***

### Objective

* Create a Jenkins job named `copy-logs`
* Fetch Apache logs from `stapp01`:
  * `/var/log/httpd/access_log`
  * `/var/log/httpd/error_log`
* Store logs on `ststor01` under:
  * `/usr/src/data`
* Schedule the job to run every 3 minutes
* Execute and verify successful log transfer

***

### Infrastructure Details

| Server Role | Hostname | User | Password |
| ------------------ | --------- | ------- | ---------- |
| Jenkins Server | jenkins | jenkins | j@rv!s |
| Application Server | stapp01 | tony | Ir0nM@n |
| Storage Server | ststor01 | natasha | Bl@kW |
| Jumphost | jump-host | thor | mjolnir123 |

***

### Step 1: SSH Key Setup (Jenkins User)

Login to Jenkins server:

```bash
ssh jenkins@jenkins
```

Switch to Jenkins environment:

```bash
sudo su - jenkins
```

Generate SSH key:

```bash
ssh-keygen -t rsa
```

Output:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa):
Created directory '/var/lib/jenkins/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:q8J4RhCrshmLM+r2qtwJcq6uqtWzPVT2eY6vaLo8Ylk jenkins@jenkins.stratos.xfusioncorp.com
```

***

### Step 2: Configure SSH Trust

Add known hosts:

```bash
ssh-keyscan stapp01 >> ~/.ssh/known_hosts
ssh-keyscan ststor01 >> ~/.ssh/known_hosts
```

Output:

```
# stapp01:22 SSH-2.0-OpenSSH_9.9
# ststor01:22 SSH-2.0-OpenSSH_9.9
```

Copy SSH keys:

```bash
ssh-copy-id tony@stapp01
ssh-copy-id natasha@ststor01
```

Output:

```
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'tony@stapp01'"

Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'natasha@ststor01'"
```

***

### Step 3: Prepare Storage Server

Login to storage server:

```bash
ssh natasha@ststor01
```

Check directory:

```bash
ls -ld /usr/src/data
```

Output:

```
ls: cannot access '/usr/src/data': No such file or directory
```

Create directory:

```bash
sudo mkdir -p /usr/src/data
sudo chown -R natasha:natasha /usr/src/data
sudo chmod 755 /usr/src/data
```

Verify:

```bash
ls /usr/src/data
```

***

### Step 4: Verify Apache Logs on Source Server

```bash
ssh tony@stapp01
```

Check logs:

```bash
sudo cat /var/log/httpd/access_log
sudo cat /var/log/httpd/error_log
```

Sample output:

```
127.0.0.1 - - [-] "GET / HTTP/1.1" 200 -
```

```
[notice] Apache/2.4 started
AH00558: httpd: Could not reliably determine server name
[core:notice] Apache/2.4.62 configured
```

***

### Step 5: Create Jenkins Job

#### Job Name

```
copy-logs
```

#### Configuration

* Type: Freestyle Project
* Build Trigger:

```
*/3 * * * *
```

***

### Build Step (Execute Shell)

```bash
ssh tony@stapp01 "sudo cat /var/log/httpd/access_log" > access_log
ssh tony@stapp01 "sudo cat /var/log/httpd/error_log" > error_log

scp access_log natasha@ststor01:/usr/src/data/
scp error_log natasha@ststor01:/usr/src/data/
```

***

### Step 6: Jenkins Job Execution

#### Manual Build Trigger

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/copy-logs
[copy-logs] $ /bin/sh -xe /tmp/jenkins13073062474580847846.sh
+ ssh tony@stapp01 sudo cat /var/log/httpd/access_log
+ ssh tony@stapp01 sudo cat /var/log/httpd/error_log
+ scp access_log natasha@ststor01:/usr/src/data/
+ scp error_log natasha@ststor01:/usr/src/data/
Finished: SUCCESS
```

***

### Step 7: Verification on Storage Server

```bash
ssh natasha@ststor01
ls -l /usr/src/data
```

Output:

```
-rw-r--r-- 1 natasha natasha  532 access_log
-rw-r--r-- 1 natasha natasha 1024 error_log
```

Check file contents:

```bash
cat /usr/src/data/access_log
```

Output:

```
127.0.0.1 - - [-] "GET / HTTP/1.1" 200 -
```

```bash
cat /usr/src/data/error_log
```

Output:

```
[notice] Apache/2.4 started
AH00558: httpd: Could not reliably determine server name
[core:notice] Apache/2.4.62 configured
```

***

### Conclusion

The Jenkins job `copy-logs` is successfully configured to collect Apache logs from `stapp01` and store them in a centralized location on `ststor01`. The job runs automatically every 3 minutes and has been validated through successful execution and log verification.

***

### Outcome

* Jenkins job created successfully
* SSH authentication configured between nodes
* Apache logs collected and transferred successfully
* Centralized storage verified
* Scheduled execution working as expected

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
