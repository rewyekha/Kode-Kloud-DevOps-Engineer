# Day 79: Jenkins Deployment Job

The Nautilus development team had a meeting with the DevOps team where they discussed automating the deployment of one of their apps using Jenkins (the one in `Stratos Datacenter`). They want to auto deploy the new changes in case any developer pushes to the repository. As per the requirements mentioned below configure the required Jenkins job.

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and `Adm!n321` password.

Similarly, you can access the `Gitea UI` using `Gitea` button. Username and password for Git are `sarah` and `Sarah_pass123`. Under user `sarah` you will find a repository named `web` that is already cloned on **App Server 1** under sarah's home (`/home/sarah/web`). `sarah` is a developer who is working on this repository.

1\. `httpd` is already installed and configured on the app server (listening on port `8080`). Ensure the `httpd` service is running on App Server 1 (e.g. start it manually if needed). You can make starting/restarting httpd part of your Jenkins job if you prefer.

2\. Create a Jenkins job named `xfusion-app-deployment` and configure it so that if anyone pushes any new change to the origin repository in `master` branch, the job should auto build and deploy the latest code on **App Server 1** under `/var/www/html` directory.\
Before deployment, ensure that the ownership of the `/var/www/html` directory is set to user `sarah`, so that Jenkins can successfully deploy files to that directory.

3\. SSH into **App Server 1** using `sarah` user credentials mentioned above. Under sarah user's home (`/home/sarah/web`) you will find a cloned Git repository named `web`. Under this repository there is an `index.html` file, update its content to `Welcome to the xFusionCorp Industries`, then push the changes to the `origin` into `master` branch. This push must trigger your Jenkins job and the latest changes must be deployed on the server, also make sure it deploys the entire repository content not only `index.html` file.

Click on the `App` button on the top bar to access the app. Please make sure the required content is loading on the main URL (e.g. http://stlb01:8091) i.e there should not be any sub-directory like http://stlb01:8091/web etc.

`Note:`\
1\. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also some times Jenkins UI gets stuck when Jenkins service restarts in the back end so in such case please make sure to refresh the UI page.

2\. Make sure Jenkins job passes even on repetitive runs as validation may try to build the job multiple times.

3\. Deployment related tasks should be done by `sudo` user on the destination server to avoid any permission issues so make sure to configure your Jenkins job accordingly.

4\. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.



## Jenkins Auto Deployment on Code Push

**Organization:** xFusionCorp Industries / Nautilus Development Team **Team:** DevOps — Stratos Datacenter **Job Type:** Jenkins Freestyle Job

***

### 1. Overview

This lab walks through configuring a Jenkins freestyle job that automatically deploys the latest code from a Gitea repository to App Server 1 whenever a developer pushes changes to the `master` branch. The deployment targets the Apache document root at `/var/www/html`, and the trigger is implemented using Poll SCM.

The end-to-end flow covers verifying the Apache service, setting up passwordless SSH from Jenkins to the app server, creating and configuring the Jenkins freestyle job, modifying application content, pushing to Git, and confirming automatic deployment.

***

### 2. Objectives

* Verify that `httpd` is running on App Server 1 and listening on port `8080`.
* Configure passwordless SSH access from the Jenkins server to the app server.
* Create a Jenkins freestyle job named `xfusion-app-deployment`.
* Configure the job to poll the `master` branch and auto-deploy on every push.
* Set ownership of `/var/www/html` to user `sarah` before each deployment.
* Update `index.html` with the required content and push to trigger the pipeline.
* Confirm the application loads correctly at the root load balancer URL.

***

### 3. Infrastructure Details

| Server         | Hostname   | User      | Password        |
| -------------- | ---------- | --------- | --------------- |
| App Server 1   | `stapp01`  | `sarah`   | `Sarah_pass123` |
| Jenkins Server | `jenkins`  | `jenkins` | `j@rv!s`        |
| Jump Host      | `jumphost` | `thor`    | `mjolnir123`    |

| Service          | Detail                            |
| ---------------- | --------------------------------- |
| Gitea Repository | `http://gitea:3000/sarah/web.git` |
| Cloned Repo Path | `/home/sarah/web` on App Server 1 |
| Deployment Path  | `/var/www/html`                   |
| Apache Port      | `8080`                            |
| App URL          | `http://stlb01:8091`              |

***

### 4. Prerequisites

#### 4.1 Required Jenkins Plugins

Navigate to:

```
Manage Jenkins → Plugins → Available Plugins
```

Install if not already present:

* Git
* Credentials
* SSH Credentials
* SSH Build Agents

> **Note:** After installing plugins, select "Restart Jenkins when installation is complete and no jobs are running." If the UI becomes unresponsive after the restart, refresh the browser.

***

### 5. Verify Apache on App Server 1

SSH into App Server 1 from the jump host:

```bash
ssh sarah@stapp01
```

Check that the `httpd` service is active:

```bash
sudo systemctl status httpd
```

Expected output:

```
httpd.service - The Apache HTTP Server
   Active: active (running)
```

Confirm it is listening on port `8080`:

```bash
sudo ss -tlnp | grep 8080
```

Expected output:

```
LISTEN 0 511 *:8080 *:* users:(("httpd",pid=1404,fd=4))
```

If the service is not running, start it:

```bash
sudo systemctl start httpd
```

***

### 6. SSH Key Setup

The Jenkins server must reach App Server 1 without a password prompt for the deploy shell script to execute non-interactively.

#### 6.1 Log into Jenkins Server

```bash
ssh jenkins@jenkins
```

Password: `j@rv!s`

#### 6.2 Generate SSH Key Pair

```bash
ssh-keygen
```

Accept the default path. This creates:

* Private key: `/var/lib/jenkins/.ssh/id_ed25519`
* Public key: `/var/lib/jenkins/.ssh/id_ed25519.pub`

#### 6.3 Copy Public Key to App Server

```bash
ssh-copy-id sarah@stapp01
```

Enter `sarah`'s password when prompted.

#### 6.4 Validate Connectivity

```bash
ssh sarah@stapp01
```

The connection must succeed without a password prompt before proceeding.

***

### 7. App Server Preparation

#### 7.1 Grant sarah Passwordless sudo

On App Server 1, open the sudoers file:

```bash
sudo visudo
```

Add the following line:

```
sarah ALL=(ALL) NOPASSWD: ALL
```

This allows the Jenkins deploy script to run `sudo` commands over SSH without interaction.

#### 7.2 Set Ownership of Deployment Directory

```bash
sudo chown -R sarah:sarah /var/www/html
```

***

### 8. Jenkins Freestyle Job Configuration

#### 8.1 Create New Job

Navigate to:

```
Jenkins Dashboard → New Item
```

| Field    | Value                    |
| -------- | ------------------------ |
| Job Name | `xfusion-app-deployment` |
| Job Type | Freestyle project        |

#### 8.2 Source Code Management

Under the job configuration, select **Git** and configure:

| Field          | Value                                      |
| -------------- | ------------------------------------------ |
| Repository URL | `http://gitea:3000/sarah/web.git`          |
| Branch         | `*/master`                                 |
| Credentials    | Username `sarah`, Password `Sarah_pass123` |

#### 8.3 Build Trigger — Poll SCM

Enable **Poll SCM** and set the schedule to poll every minute:

```
* * * * *
```

This causes Jenkins to check the repository for new commits every minute. When a push is detected, a build is triggered automatically.

#### 8.4 Build Step — Execute Shell

Add a build step of type **Execute shell** and paste the following script:

```bash
ssh sarah@stapp01 "sudo chown -R sarah:sarah /var/www/html"

rm -rf web

git clone http://sarah:Sarah_pass123@gitea:3000/sarah/web.git

scp -r web/* sarah@stapp01:/var/www/html/

ssh sarah@stapp01 "sudo systemctl restart httpd"
```

Save the job configuration.

***

### 9. Build Script Walkthrough

#### 9.1 Ownership Reset

```bash
ssh sarah@stapp01 "sudo chown -R sarah:sarah /var/www/html"
```

Ensures `sarah` owns the deployment directory before every run. This prevents permission errors on repeated builds.

#### 9.2 Clean Clone

```bash
rm -rf web
git clone http://sarah:Sarah_pass123@gitea:3000/sarah/web.git
```

Removes any previously cloned copy in the Jenkins workspace and performs a fresh clone. This ensures the full repository content is deployed, not just modified files.

#### 9.3 Deploy to App Server

```bash
scp -r web/* sarah@stapp01:/var/www/html/
```

Copies the entire repository content to the Apache document root over SSH. Using `web/*` ensures no subdirectory is created under `/var/www/html`.

#### 9.4 Restart Apache

```bash
ssh sarah@stapp01 "sudo systemctl restart httpd"
```

Restarts the `httpd` service after deployment to ensure the server picks up any updated content.

***

### 10. Update Application Content and Push

SSH into App Server 1:

```bash
ssh sarah@stapp01
```

Navigate to the cloned repository:

```bash
cd /home/sarah/web
```

Update the index file:

```bash
echo "Welcome to the xFusionCorp Industries" > index.html
```

Stage and commit the change:

```bash
git add .
git commit -m "Update index.html with required content"
```

Push to the remote `master` branch:

```bash
git push origin master
```

Expected output:

```
Enumerating objects: 5, done.
Writing objects: 100% (3/3), 285 bytes | 285.00 KiB/s, done.
To http://gitea:3000/sarah/web.git
   c373cc2..527c67d  master -> master
```

***

### 11. Automatic Build Trigger

Within one minute of the push, Jenkins detects the new commit via Poll SCM and executes the build automatically. No manual trigger is required.

To monitor the build, navigate to the job page in Jenkins and observe the build history and console output.

***

### 12. Common Errors and Resolutions

| Error                                                   | Cause                                         | Resolution                                                                           |
| ------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------ |
| `Permission denied` when copying to `/var/www/html`     | Directory not owned by `sarah`                | Run `sudo chown -R sarah:sarah /var/www/html` on the app server                      |
| `Host key verification failed` in shell step            | Jenkins has not connected to `stapp01` before | Manually run `ssh sarah@stapp01` from the Jenkins server once to accept the host key |
| Content loads under `/web` subdirectory instead of root | `scp` target path included the repo directory | Use `web/*` in the `scp` command rather than `web/`                                  |
| Build passes but content not updated                    | Apache not restarted after deploy             | Add `sudo systemctl restart httpd` to the end of the shell script                    |
| Job does not trigger after push                         | Poll SCM schedule not saved or incorrect      | Verify the schedule is `* * * * *` and the job configuration is saved                |
| Jenkins UI unresponsive after plugin install            | Service restarting in the background          | Wait and refresh the browser                                                         |

***

### 13. Verification

#### 13.1 Confirm Deployment Content

Open the application URL in a browser:

```
http://stlb01:8091
```

The page must display:

```
Welcome to the xFusionCorp Industries
```

The content must load at the root path. No subdirectory such as `/web` should appear in the URL.

#### 13.2 Confirm Repeated Build Stability

The job must pass on repeated runs without manual intervention. The ownership reset and clean clone in the build script ensure idempotent behavior across multiple executions.

***

### 14. Summary

This lab demonstrates a complete CI/CD setup using a Jenkins freestyle job with Poll SCM as the trigger mechanism. The pipeline clones the Gitea repository fresh on every build, deploys the full content to the Apache document root via SCP, and restarts the web server — all over SSH from the Jenkins server to the app server.

Key reliability considerations include resetting directory ownership before each deployment, performing a clean clone rather than an incremental pull, and restarting Apache after every deploy. These steps ensure the job produces consistent results whether it runs once or many times in succession.



<figure><img src="../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

.

<figure><img src="../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

.

<figure><img src="../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>
