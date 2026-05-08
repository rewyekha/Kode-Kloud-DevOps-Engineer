# Day 75: Jenkins Slave Nodes

The Nautilus DevOps team has installed and configured new Jenkins server in Stratos DC which they will use for CI/CD and for some automation tasks. There is a requirement to add all app servers as slave nodes in Jenkins so that they can perform tasks on these servers using Jenkins. Find below more details and accomplish the task accordingly.

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.\
\
1\. Add all app servers as SSH build agent/slave nodes in Jenkins. Slave node name for `app server 1`, `app server 2` and `app server 3` must be `App_server_1`, `App_server_2`, `App_server_3` respectively.\
\
2\. Add labels as below:\
\
`App_server_1 : stapp01`\
\
`App_server_2 : stapp02`\
\
`App_server_3 : stapp03`\
\
3\. Remote root directory for `App_server_1` must be `/home/tony/jenkins`, for `App_server_2` must be `/home/steve/jenkins` and for `App_server_3` must be `/home/banner/jenkins`.\
4\. Make sure slave nodes are online and working properly.

`Note:`

1\. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case, please make sure to refresh the UI page.\
\
2\. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.

## Jenkins: Adding Application Servers as SSH Agent Nodes

> This document covers the complete procedure for adding all Nautilus application servers as SSH-based Jenkins agent nodes on the newly installed Jenkins server in Stratos DC.

***

### Overview

This runbook documents the procedure followed to successfully add all Nautilus application servers as **SSH-based Jenkins agent (slave) nodes**. These agents execute CI/CD jobs and automation tasks directly on the application servers.

The setup covers credential management, node configuration, Java compatibility resolution, and final verification.

***

### Scope

The following application servers are configured as Jenkins agents:

| Node Name      | Hostname | Label   | Remote User |
| -------------- | -------- | ------- | ----------- |
| App\_server\_1 | stapp01  | stapp01 | tony        |
| App\_server\_2 | stapp02  | stapp02 | steve       |
| App\_server\_3 | stapp03  | stapp03 | banner      |

***

### Infrastructure Reference

| Server               | Hostname | User   | Purpose        |
| -------------------- | -------- | ------ | -------------- |
| Application Server 1 | stapp01  | tony   | Nautilus App 1 |
| Application Server 2 | stapp02  | steve  | Nautilus App 2 |
| Application Server 3 | stapp03  | banner | Nautilus App 3 |
| Jenkins Server       | jenkins  | admin  | CI/CD          |

***

### Step 1 — Verify Java on Application Servers

Each agent requires Java installed to run `remoting.jar`. The Java version on agents must match the Jenkins controller.

Jenkins controller runs Java 17. Agents with Java 11 will throw `UnsupportedClassVersionError` (class version 61 vs 55). &#x20;

#### Resolution

Install Java 17 on all application servers and set it as the default runtime.

**Verify the Java version:**

```bash
java -version
```

**Expected output:**

```
openjdk version "17"
```

***

### Step 2 — Configure Jenkins Credentials

Create separate SSH credentials for each application server.

**Jenkins path:**

```
Manage Jenkins → Credentials → System → Global
```

**Credential type:** Username with password

| Credential ID   | User   | Server  |
| --------------- | ------ | ------- |
| `ssh-tony-id`   | tony   | stapp01 |
| `ssh-steve-id`  | steve  | stapp02 |
| `ssh-banner-id` | banner | stapp03 |

***

### Step 3 — Install Required Jenkins Plugin

The **SSH Build Agents Plugin** is required to enable SSH-based agent connections.

**Installation path:**

```
Manage Jenkins → Plugins → Available / Installed
```

***

### Step 4 — Add Jenkins Nodes (Agents)

Nodes are added as **Permanent Agents** using SSH.

#### Common Settings

| Field                 | Value                               |
| --------------------- | ----------------------------------- |
| Launch method         | Launch agents via SSH               |
| Host key verification | Non-verifying Verification Strategy |
| Executors             | 1                                   |
| Usage                 | Use this node as much as possible   |

***

#### App\_server\_1

| Field                 | Value                |
| --------------------- | -------------------- |
| Node Name             | App\_server\_1       |
| Hostname              | stapp01              |
| Credentials           | `ssh-tony-id`        |
| Label                 | stapp01              |
| Remote Root Directory | `/home/tony/jenkins` |

***

#### App\_server\_2

| Field                 | Value                 |
| --------------------- | --------------------- |
| Node Name             | App\_server\_2        |
| Hostname              | stapp02               |
| Credentials           | `ssh-steve-id`        |
| Label                 | stapp02               |
| Remote Root Directory | `/home/steve/jenkins` |

***

#### App\_server\_3

| Field                 | Value                  |
| --------------------- | ---------------------- |
| Node Name             | App\_server\_3         |
| Hostname              | stapp03                |
| Credentials           | `ssh-banner-id`        |
| Label                 | stapp03                |
| Remote Root Directory | `/home/banner/jenkins` |

***

### Troubleshooting

#### Issue 1 — Java version mismatch

**Error:**

```
UnsupportedClassVersionError (class version 61 vs 55)
```

**Cause:** Jenkins controller compiled with Java 17 while the agent was running Java 11.

**Resolution:**

1. Install Java 17 on all agent servers.
2. Set Java 17 as the default runtime on each agent.
3. Verify with `java -version`.

***

### Notes

* Screenshots of credentials, node configuration, and agent online status should be captured for audit and review.
* This setup supports scalable, label-based CI/CD job execution across application servers.

***

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>



Install Java on all three machines

```bash
sudo yum install -y java-17-openjdk java-17-openjdk-devel
```

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Relaunch the Nodes



<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
