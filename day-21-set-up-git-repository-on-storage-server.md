# Day 21: Set Up Git Repository on Storage Server
The Nautilus development team has provided requirements to the DevOps team for a new application development project, specifically requesting the establishment of a Git repository. Follow the instructions below to create the Git repository on the Storage server in the Stratos DC:

```
Utilize yum to install the git package on the Storage Server.

Create a bare repository named /opt/games.git (ensure exact name usage).
```

## Git Repository Setup on Nautilus Storage Server
### Overview
This document describes the steps followed to install Git and create a bare Git repository on the Nautilus **Storage Server (`ststor01`)** as requested by the development team.

***

### Infrastructure Details
* **Server Name:** ststor01
* **Hostname:** ststor01.stratos.xfusioncorp.com
* **Purpose:** Nautilus Storage Server
* **User:** natasha

***

### Step 1: Connect to the Storage Server
SSH into the Storage Server from the jump host.

```bash
thor@jumphost ~$ ssh natasha@ststor01.stratos.xfusioncorp.com
```

Terminal output:

```
The authenticity of host 'ststor01.stratos.xfusioncorp.com (172.16.238.15)' can't be established.
ED25519 key fingerprint is SHA256:XaAJ4atJiIY9mgPUdpsWKI3F96px4wQXhyR9miyGmTM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
natasha@ststor01.stratos.xfusioncorp.com's password:
```

Successful login:

```
[natasha@ststor01 ~]$
```

***

### Step 2: Install Git Using yum
Install Git on the Storage Server using the `yum` package manager.

```bash
sudo yum install -y git
```

System message and authentication:

```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for natasha:
```

Package installation output (truncated for brevity):

```
Dependencies resolved.
===========================================================================================================================
Installing:
 git                                  x86_64               2.52.0-1.el9                      appstream
Installing dependencies:
 git-core                             x86_64               2.52.0-1.el9
 git-core-doc                         noarch               2.52.0-1.el9
 ...
Transaction Summary
===========================================================================================================================
Install  67 Packages

Total download size: 17 M
Installed size: 72 M
...
Complete!
```

***

### Step 3: Verify Git Installation
Confirm that Git was installed successfully.

```bash
git --version
```

Output:

```
git version 2.52.0
```

***

### Step 4: Create a Bare Git Repository
Create a bare Git repository at the required location.

```bash
sudo git init --bare /opt/games.git
```

Output:

```
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0.
...
Initialized empty Git repository in /opt/games.git/
```

> **Note:** The branch name warning is informational only and does not impact repository creation.

***

### Step 5: Verify Repository Creation
Confirm that the bare repository exists and verify its permissions.

```bash
ls -ld /opt/games.git
```

Output:

```
drwxr-xr-x 6 root root 4096 Feb 10 04:37 /opt/games.git
```

***

### Final Status
 Git installed successfully on the Storage Server
 Bare Git repository created at `/opt/games.git`
 Repository name and path match the project requirements
