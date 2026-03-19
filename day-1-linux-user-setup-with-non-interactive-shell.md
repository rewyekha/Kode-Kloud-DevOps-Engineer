# Day 1: Linux User Setup with Non-Interactive Shell
To accommodate the backup agent tool's specifications, the system admin team at `xFusionCorp Industries` requires the creation of a user with a non-interactive shell.

Create a user named `mark` with a non-interactive shell on `App Server 1`.

> Note: You can find the infrastructure details by clicking on the **Details of all Users and Servers** button on the top-right section of the page.

### Project Nautilus
#### OVERVIEW
Project Nautilus is run by the Naval subdivision within xFusionCorp Industries. The Nautilus Application helps the Naval forces to make smart procurement decisions on manned and unmanned maritime systems while ensuring that operational requirements are met. It aims to provide best in class operational support, improve the safety and life extension of existing machines, and reduce cost of ownership.

#### Current Repertoire
1. Sonar Technology and Systems
2. LUSV - Large Unmanned Surface Vehicles
3. Autonomous Unmanned Undersea Pods
4. Nuclear Submarines
5. Laser Guidance Systems

#### Application Architecture
Nautilus deployment architecture can be viewed [here](https://www.lucidchart.com/documents/edit/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0?shared=true)

The Nautilus is a three-tier application and is deployed in the Stratos Datacenter in the North America Region.

* **Data Tier:** The Data tier is the layer that stores data with the retrieval storage and execution methods made by the application layer. We are making use of MariaDB which is one of the most popular open source relational databases.
* **Application Tier:** Makes use of a LAMP which is a stack of open-source software that can be used to create web applications. LAMP is an acronym that usually consists of the Linux OS, the Apache HTTP Server, a MySQL relational DBMS (like MariaDB), and PHP.
* **Client Tier:** The application client which in this case is a web browser software that processes and displays HTML resources, issues HTTP requests for resources, and processes HTTP responses.
* **Load Balancer:** Nginx is used for HTTP Load Balancing to distribute requests through multiple application servers.

#### Shared Services
* **Storage Filer:** A NAS (Network Attached Storage) filer is used to provide reliable and stable external storage for the application tier servers.
* **SFTP Server:** SFTP, which stands for SSH File Transfer Protocol is used to transfer data amongst two remote systems.
* **Backup Server:** A staging backup system used for short term archival.
* **Jump Server:** The intermediary host or an SSH gateway to a remote network hosting the Nautilus application.

#### Infrastructure Details
| **Server Name** | **IP**        | **Hostname**                       | **User** | **Password** | **Purpose**                    |
| --------------- | ------------- | ---------------------------------- | -------- | ------------ | ------------------------------ |
| stapp01         | 172.16.238.10 | stapp01.stratos.xfusioncorp.com    | tony     | Ir0nM@n      | Nautilus App 1                 |
| stapp02         | 172.16.238.11 | stapp02.stratos.xfusioncorp.com    | steve    | Am3ric@      | Nautilus App 2                 |
| stapp03         | 172.16.238.12 | stapp03.stratos.xfusioncorp.com    | banner   | BigGr33n     | Nautilus App 3                 |
| stlb01          | 172.16.238.14 | stlb01.stratos.xfusioncorp.com     | loki     | Mischi3f     | Nautilus HTTP LBR              |
| stdb01          | 172.16.239.10 | stdb01.stratos.xfusioncorp.com     | peter    | Sp!dy        | Nautilus DB Server             |
| ststor01        | 172.16.238.15 | ststor01.stratos.xfusioncorp.com   | natasha  | Bl@kW        | Nautilus Storage Server        |
| stbkp01         | 172.16.238.16 | stbkp01.stratos.xfusioncorp.com    | clint    | H@wk3y3      | Nautilus Backup Server         |
| stmail01        | 172.16.238.17 | stmail01.stratos.xfusioncorp.com   | groot    | Gr00T123     | Nautilus Mail Server           |
| jump\_host      | Dynamic       | jump\_host.stratos.xfusioncorp.com | thor     | mjolnir123   | Jump Server to Access Stork DC |
| jenkins         | 172.16.238.19 | jenkins.stratos.xfusioncorp.com    | jenkins  | j@rv!s       | Jenkins Server for CI/CD       |

```
thor@jumphost ~$ ssh tony@172.16.238.10
The authenticity of host '172.16.238.10 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:8tonJP761VoH5SfGpmbvGUN0ccm+QMNcKgQQM/7djzQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.10' (ED25519) to the list of known hosts.
tony@172.16.238.10's password:
[tony@stapp01 ~]$ sudo useradd -m -s /sbin/nologin mark

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
[tony@stapp01 ~]$ sudo passwd mark
Changing password for user mark.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
Sorry, passwords do not match.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[tony@stapp01 ~]$ grep "^mark:" /etc/passwd
mark:x:1002:1002::/home/mark:/sbin/nologin
[tony@stapp01 ~]$
```

#### **Step-by-Step Instructions**
**1) SSH into App Server 1**

Use the provided credentials for `stapp01`:

```
ssh tony@172.16.238.10
```

Password:

```
Ir0nM@n
```

***

**2) Create the user `mark` with a non-interactive shell**

Most Linux systems use `/sbin/nologin` or `/usr/sbin/nologin` for non-interactive shells. Run:

```bash
sudo useradd -m -s /sbin/nologin mark
```

If `/sbin/nologin` doesn’t exist, try:

```bash
sudo useradd -m -s /usr/sbin/nologin mark
```

Explanation:

| Option             | Meaning                                         |
| ------------------ | ----------------------------------------------- |
| `-m`               | Create home directory `/home/mark`              |
| `-s /sbin/nologin` | Assign a non-interactive shell (no login shell) |

***

**3) (Optional) Set a password for `mark`**

If you need the account to have a password (even with no login shell), run:

```bash
sudo passwd mark
```

Then enter the desired password.

If the account should **never login at all**, _do not assign a password_.

***

**4) Verify the user was created properly**

Check the entry in `/etc/passwd`:

```bash
grep "^mark:" /etc/passwd
```

You should see something like:

```
mark:x:1001:1001::/home/mark:/sbin/nologin
```

The last field (`/sbin/nologin`) confirms the _non-interactive shell_.

***

#### Additional Notes
Users with `/sbin/nologin` **cannot log in interactively**, but can still be used by services (like your backup agent).
If your backup tool needs a specific shell path (sometimes `/bin/false`), you can use that instead:

```bash
sudo useradd -m -s /bin/false mark
```

***

#### Summary
**Command to use on App Server 1 (`stapp01`):**

```bash
sudo useradd -m -s /sbin/nologin mark
```

(optional)

```bash
sudo passwd mark
```
