# Day 78: Jenkins Conditional Pipeline

The development team of xFusionCorp Industries is working on to develop a new static website and they are planning to deploy the same on Nautilus App Server using Jenkins pipeline. They have shared their requirements with the DevOps team and accordingly we need to create a Jenkins pipeline job. Please find below more details about the task:

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.

Similarly, click on the `Gitea` button on the top bar to access the Gitea UI. Login using username `sarah` and password `Sarah_pass123`. There under user `sarah` you will find a repository named `web_app` that is already cloned on **App Server 1** under `/var/www/html`. sarah is a developer who is working on this repository.

1. Add a slave node named `App Server 1`. It should be labeled as `stapp01` and its remote root directory should be `/home/sarah/jenkins_agent` (the repository is cloned under `/var/www/html`).
2. We have already cloned repository on **App Server 1** under `/var/www/html`.
3. Apache is already installed on the app server and is running on port `8080`.
4. Create a Jenkins pipeline job named `devops-webapp-job` (it must not be a `Multibranch pipeline`) and configure it to:
   * Add a string parameter named `BRANCH`.
   * It should conditionally deploy the code from `web_app` repository under `/var/www/html` on **App Server 1**, as this is the document root of the app server. The pipeline should have a single stage named `Deploy` ( which is case sensitive ) to accomplish the deployment.
   * The pipeline should be conditional, if the value `master` is passed to the `BRANCH` parameter then it must deploy the `master` branch, on the other hand if the value `feature` is passed to the `BRANCH` parameter then it must deploy the `feature` branch.

LB server is already configured. You should be able to see the latest changes you made by clicking on the `App` button. Please make sure the required content is loading on the main URL `https://<LBR-URL>` i.e there should not be a sub-directory like `https://<LBR-URL>/web_app` etc.\
\
`Note:`

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.
2. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.



## Jenkins Pipeline Deployment

**Organization:** xFusionCorp Industries **Team:** DevOps **Scope:** Static website deployment via Jenkins Pipeline to Nautilus App Server 1

***

### 1. Overview

This lab walks through configuring a Jenkins pipeline job that deploys a static website from a Gitea repository to a remote app server. The pipeline accepts a branch name as a parameter and conditionally checks out and pulls either the `master` or `feature` branch into the Apache document root on App Server 1.

The setup covers adding a Jenkins SSH agent node, configuring SSH key-based authentication between the Jenkins server and the app server, and writing a parameterized conditional pipeline script.

***

### 2. Objectives

* Register App Server 1 as a Jenkins SSH agent node labeled `stapp01`.
* Configure passwordless SSH access from the Jenkins server to the app server.
* Create a parameterized Jenkins pipeline job named `devops-webapp-job`.
* Implement conditional deployment logic based on the value of the `BRANCH` parameter.
* Verify that the correct branch content is served at the root load balancer URL.

***

### 3. Environment Details

#### Access

| Service | URL                  | Username | Password        |
| ------- | -------------------- | -------- | --------------- |
| Jenkins | Jenkins UI (top bar) | `admin`  | `Adm!n321`      |
| Gitea   | Gitea UI (top bar)   | `sarah`  | `Sarah_pass123` |

#### Infrastructure

| Component       | Detail                                        |
| --------------- | --------------------------------------------- |
| App Server      | `stapp01` — IP `10.244.235.15`                |
| App Server OS   | RHEL / CentOS Stream 9                        |
| Jenkins Server  | `jenkins` — IP `10.244.29.237` (Ubuntu 24.04) |
| Repository      | `sarah/web_app` on Gitea                      |
| Deployment Path | `/var/www/html`                               |
| Apache Port     | `8080`                                        |
| Agent Root      | `/home/sarah/jenkins_agent`                   |

***

### 4. Prerequisites

#### 4.1 Required Jenkins Plugins

Ensure the following plugins are installed before proceeding. Navigate to:

```
Manage Jenkins → Plugins → Available Plugins
```

Install if not already present:

* Pipeline
* Git
* Credentials
* SSH Credentials
* SSH Build Agents

> **Note:** After installing plugins, select "Restart Jenkins when installation is complete and no jobs are running" on the update centre page. If the UI becomes unresponsive after the restart, refresh the browser.

#### 4.2 App Server Dependencies

Install Java 17 and `screen` on App Server 1. These are required for the Jenkins agent process to run.

SSH into the app server from the jump host:

```bash
ssh sarah@stapp01
```

Then install the required packages:

```bash
sudo yum install -y java-17-openjdk screen
```

***

### 5. SSH Key Setup

The Jenkins server must be able to connect to App Server 1 without a password prompt. This is achieved by generating an RSA key pair on the Jenkins server and copying the public key to the app server.

#### 5.1 Generate SSH Key Pair on Jenkins Server

Log into the Jenkins server from the jump host:

```bash
ssh jenkins@jenkins
```

Generate the key pair:

```bash
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

This creates:

* Private key: `/var/lib/jenkins/.ssh/id_rsa`
* Public key: `/var/lib/jenkins/.ssh/id_rsa.pub`

#### 5.2 Copy Public Key to App Server

```bash
ssh-copy-id sarah@stapp01
```

Enter `sarah`'s password when prompted. The public key is appended to `/home/sarah/.ssh/authorized_keys` on the app server.

#### 5.3 Validate Connectivity

```bash
ssh sarah@stapp01
```

The connection should succeed without a password prompt.

***

### 6. App Server Preparation

#### 6.1 Fix Repository Ownership

The Jenkins agent runs as `sarah`. The deployment directory must be owned by this user to allow git operations without permission errors.

```bash
sudo chown -R sarah:sarah /var/www/html
```

***

### 7. Jenkins Agent Node Configuration

#### 7.1 Add New Node

Navigate to:

```
Manage Jenkins → Nodes → New Node
```

#### 7.2 Node Parameters

| Parameter                      | Value                                                                  |
| ------------------------------ | ---------------------------------------------------------------------- |
| Node Name                      | `App Server 1`                                                         |
| Type                           | Permanent Agent                                                        |
| Remote Root Directory          | `/home/sarah/jenkins_agent`                                            |
| Labels                         | `stapp01`                                                              |
| Launch Method                  | Launch agents via SSH                                                  |
| Host                           | `stapp01`                                                              |
| Credentials                    | SSH with username `sarah` and the private key generated in Section 5.1 |
| Host Key Verification Strategy | Non verifying (or manually trusted)                                    |

#### 7.3 Add SSH Credentials in Jenkins

Navigate to:

```
Manage Jenkins → Credentials → System → Global credentials → Add Credentials
```

| Field       | Value                                            |
| ----------- | ------------------------------------------------ |
| Kind        | SSH Username with private key                    |
| Username    | `sarah`                                          |
| Private Key | Paste contents of `/var/lib/jenkins/.ssh/id_rsa` |

Save and assign these credentials to the node configuration.

***

### 8. Pipeline Job Configuration

#### 8.1 Create New Job

Navigate to:

```
Jenkins Dashboard → New Item
```

| Field    | Value                               |
| -------- | ----------------------------------- |
| Job Name | `devops-webapp-job`                 |
| Job Type | Pipeline (not Multibranch Pipeline) |

#### 8.2 Add String Parameter

Under the job configuration, enable:

```
This project is parameterized
```

Add a String Parameter:

| Field         | Value    |
| ------------- | -------- |
| Name          | `BRANCH` |
| Default Value | `master` |

#### 8.3 Pipeline Script

Paste the following script into the Pipeline definition section:

```groovy
pipeline {
    agent { label 'stapp01' }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master')
    }

    stages {
        stage('Deploy') {
            steps {
                script {

                    if (params.BRANCH == "master") {

                        sh '''
                        git config --global --add safe.directory /var/www/html
                        cd /var/www/html
                        git reset --hard
                        git clean -fd
                        git checkout master
                        git pull origin master
                        '''

                    } else if (params.BRANCH == "feature") {

                        sh '''
                        git config --global --add safe.directory /var/www/html
                        cd /var/www/html
                        git reset --hard
                        git clean -fd
                        git checkout feature
                        git pull origin feature
                        '''

                    } else {

                        error("Invalid branch name")

                    }
                }
            }
        }
    }
}
```

Save the job configuration.

***

### 9. Pipeline Script Walkthrough

#### 9.1 Agent Selection

```groovy
agent { label 'stapp01' }
```

Directs Jenkins to run all pipeline stages on the node labeled `stapp01`, which corresponds to App Server 1.

#### 9.2 Parameter Declaration

```groovy
parameters {
    string(name: 'BRANCH', defaultValue: 'master')
}
```

Declares a string parameter that is passed at build time. The default value is `master`.

#### 9.3 Conditional Deployment Logic

The `Deploy` stage evaluates the `BRANCH` parameter and runs the appropriate shell block. If neither `master` nor `feature` is passed, the pipeline fails with an explicit error message.

```groovy
if (params.BRANCH == "master") {
    // deploy master
} else if (params.BRANCH == "feature") {
    // deploy feature
} else {
    error("Invalid branch name")
}
```

#### 9.4 Git Cleanup Before Checkout

```groovy
git reset --hard
git clean -fd
```

These commands discard any local modifications or untracked files before switching branches. This prevents checkout failures caused by uncommitted changes in the working directory.

#### 9.5 Safe Directory Configuration

```groovy
git config --global --add safe.directory /var/www/html
```

Resolves the `fatal: detected dubious ownership in repository` error that occurs when the git process user does not match the directory owner.

***

### 10. Running the Pipeline

#### 10.1 Deploy Master Branch

Navigate to the job and select:

```
Build with Parameters
```

Set:

```
BRANCH = master
```

#### 10.2 Deploy Feature Branch

Run the job again with:

```
BRANCH = feature
```

***

### 11. Build Log Reference

#### Successful Master Branch Build

```
Running on App Server 1 in /home/sarah/jenkins_agent/workspace/devops-webapp-job
[Pipeline] { (Deploy)
+ git config --global --add safe.directory /var/www/html
+ cd /var/www/html
+ git reset --hard
HEAD is now at d943ea7 Added feature.html file
+ git clean -fd
+ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
+ git pull origin master
Already up to date.
Finished: SUCCESS
```

#### Successful Feature Branch Build

```
Running on App Server 1 in /home/sarah/jenkins_agent/workspace/devops-webapp-job
[Pipeline] { (Deploy)
+ git config --global --add safe.directory /var/www/html
+ cd /var/www/html
+ git reset --hard
HEAD is now at bd7e292 Added index.html file
+ git clean -fd
+ git checkout feature
Switched to branch 'feature'
+ git pull origin feature
Already up to date.
Finished: SUCCESS
```

***

### 12. Common Errors and Resolutions

| Error                                                  | Cause                                               | Resolution                                                                          |
| ------------------------------------------------------ | --------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `fatal: detected dubious ownership in repository`      | Git process user differs from directory owner       | Add `git config --global --add safe.directory /var/www/html` before any git command |
| `unable to unlink old 'index.html': Permission denied` | `sarah` does not own `/var/www/html`                | Run `sudo chown -R sarah:sarah /var/www/html` on the app server                     |
| `Your local changes would be overwritten by checkout`  | Uncommitted modifications exist in the working tree | Run `git reset --hard` and `git clean -fd` before switching branches                |
| Jenkins agent fails to connect                         | SSH key not copied or credentials misconfigured     | Re-run `ssh-copy-id sarah@stapp01` and verify credentials in Jenkins                |
| Jenkins UI unresponsive after plugin install           | Service restarting in the background                | Wait and refresh the browser                                                        |

***

### 13. Verification

#### 13.1 Confirm Active Branch on App Server

```bash
cd /var/www/html
git branch
```

Expected output after a feature branch deployment:

```
* feature
  master
```

#### 13.2 Confirm Application Loads

Open the load balancer URL in a browser:

```
https://<LBR-URL>
```

The application must load at the root path. No subdirectory such as `/web_app` should be required.

***

### 14. Summary

This lab covers the end-to-end setup of a Jenkins SSH agent and a parameterized conditional pipeline for branch-based static site deployment. The key steps are establishing passwordless SSH connectivity between Jenkins and the app server, registering the app server as a labeled agent node, and writing a pipeline script that conditionally deploys either the `master` or `feature` branch based on a runtime parameter.

The git cleanup commands (`reset --hard`, `clean -fd`) and the safe directory configuration are necessary operational details that prevent common failure modes when deploying to a shared directory outside the Jenkins workspace.

<figure><img src="../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>
