# Day 74: Jenkins Database Backup Job

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

There is a requirement to create a Jenkins job to automate the database backup. Below you can find more details to accomplish this task:

Click on the `Jenkins` button on the top bar to access the Jenkins UI. Login using username `admin` and password `Adm!n321`.

1. Create a Jenkins job named `database-backup`.
2. Configure it to take a database dump of the `kodekloud_db01` database present on the **App server (stapp01)** in Stratos Datacenter, the database user is `kodekloud_roy` and password is `asdfgdsd`.
3. The dump should be named in `db_$(date +%F).sql` format, where `date +%F` is the current date.
4. Copy the `db_$(date +%F).sql` dump to the **Storage server (ststor01)** under location `/home/natasha/db_backups`.
5. Further, schedule this job to run periodically at `*/10 * * * *` (please use this exact schedule format).

`Note:`

1. You might need to install some plugins and restart Jenkins service. So, we recommend clicking on `Restart Jenkins when installation is complete and no jobs are running` on plugin installation/update page i.e `update centre`. Also, Jenkins UI sometimes gets stuck when Jenkins service restarts in the back end. In this case please make sure to refresh the UI page.
2. Please make sure to define you cron expression like this `*/10 * * * *` (this is just an example to run job every 10 minutes).
3. For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.<br>

***

## Jenkins Automated Database Backup – Implementation Guide

### Document Purpose

***

### Problem Statement

#### Requirement

Create a Jenkins job to automate a database backup with the following specifications:

1. Create a Jenkins job named `database-backup`.
2. Take a MySQL database dump of:
   * Database: `kodekloud_db01`
   * Database user: `kodekloud_roy`
   * Database password: `asdfgdsd`
3. The database is hosted on:
   * Application Server: `stapp01`
4. Name the dump file using the format: db\_$(date +%F).sql
5. Copy the dump file to: /home/natasha/db\_backups on the Storage Server `ststor01`.
6. Schedule the Jenkins job to run periodically using the cron expression: \*/10 \* \* \* \*

***

### Infrastructure Details

| Server Role        | Hostname  | User    | Purpose                        |
| ------------------ | --------- | ------- | ------------------------------ |
| Jump Host          | jump-host | thor    | Secure access                  |
| Jenkins Server     | jenkins   | jenkins | CI/CD automation               |
| Application Server | stapp01   | tony    | Hosts application and database |
| Storage Server     | ststor01  | natasha | Stores backup data             |

***

### Solution Overview

The solution uses:

* Jenkins Freestyle Job
* SSH-based automation
* MySQL `mysqldump`
* Passwordless SSH authentication
* Cron-based job scheduling

***

### Implementation Steps

#### Step 1: Access Jenkins Server

From the jump host, establish an SSH session to the Jenkins server:

ssh jenkins@jenkins\`\`

Expected output:

The authenticity of host 'jenkins (10.244.73.168)' can't be established.ED25519 key fingerprint is SHA256:tyX2fFBjS0A5pNlRaih+266sGG9JD0LpsFdIAX9iMzA.Are you sure you want to continue connecting (yes/no/\[fingerprint])? yesWarning: Permanently added 'jenkins' (ED25519) to the list of known hosts.jenkins@jenkins's password:Welcome to Ubuntu 24.04.4 LTS

***

#### Step 2: Generate SSH Key for Jenkins User

`ssh-keygen -t rsa -b 2048`

Terminal output:

```bash
Created directory '/var/lib/jenkins/.ssh'.Your identification has been saved in /var/lib/jenkins/.ssh/id_rsaYour public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
```

This key enables passwordless SSH from Jenkins.

***

#### Step 3: Copy SSH Key to Application Server

```bash
ssh-copy-id tony@stapp01
```

Output:

```bash
The authenticity of host 'stapp01 (10.244.240.136)' can't be established.
Are you sure you want to continue connecting? yes
tony@stapp01's password:Number of key(s) added: 1
```

***

#### Step 4: Copy SSH Key to Storage Server

ssh-copy-id natasha@ststor01

Output:

The authenticity of host 'ststor01 (10.244.29.238)' can't be established.Are you sure you want to continue connecting? yesnatasha@ststor01's password:Number of key(s) added: 1

***

#### Step 5: Verify Storage Path and Permissions

ssh natasha@ststor01

ls -lh /home/natasha

drwxr-xr-x 2 natasha natasha 4096 Apr 17 04:08 db\_backups

Ensure permissions are correct:

chmod 755 /home/natasha/db\_backups

***

#### Step 6: Jenkins Job Configuration

**Job Type**

* Freestyle Project

**Job Name**

```
database-backup
```

**Build Trigger (Cron Schedule)**

\*/10 \* \* \* \*

**Build Step (Shell Script)**

\#!/bin/bash\
DATE=$(date +%F)DUMP\_FILE=db\_${DATE}.sql\
ssh tony@stapp01 << EOFmysqldump -u kodekloud\_roy -pasdfgdsd kodekloud\_db01 > /tmp/${DUMP\_FILE}EOF\
scp tony@stapp01:/tmp/${DUMP\_FILE} natasha@ststor01:/home/natasha/db\_backups/\
ssh tony@stapp01 "rm -f /tmp/${DUMP\_FILE}"

***

### Job Execution Output

Triggered manually using **Build Now**.

#### Jenkins Console Output

Started by user adminRunning as SYSTEMBuilding in workspace /var/lib/jenkins/workspace/database-backup\[database-backup] $ /bin/bash /tmp/jenkins17607366669267494748.shPseudo-terminal will not be allocated because stdin is not a terminal.Finished: SUCCESS

***

### Backup Verification

On the storage server:

ls -l /home/natasha/db\_backups

Output:

-rw-r--r-- 1 natasha natasha 1319 Apr 17 04:17 db\_2026-04-17.sql

This confirms:

* Backup file exists
* Filename follows date-based format
* Database dump completed successfully

***

### Outcome Summary

* Jenkins job executed without errors
* Database dump created on application server
* Backup securely copied to storage server
* Job scheduled to run every 10 minutes
* Passwordless SSH ensured reliable automation

***

### Conclusion

The Jenkins-based automation for database backup has been implemented and validated successfully.

***

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

***

[_Reyas Khan_](https://reyaskhan.me) _|_ [_GitHub_](https://github.com/rewyekha)
