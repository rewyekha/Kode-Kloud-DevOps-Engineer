# Day 7: Linux SSH Authentication

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The system admins team of `xFusionCorp Industries` has set up some scripts on `jump host` that run on regular intervals and perform operations on all app servers in `Stratos Datacenter`. To make these scripts work properly we need to make sure the `thor` user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e `tony` for app server 1). Based on the requirements, perform the following:

Set up a password-less authentication from user `thor` on jump host to all app servers through their respective sudo users.

### Objective

Enable **password-less SSH access** from:

* **User:** `thor`
* **Host:** `jump_host`

to **all app servers**, using their **respective sudo users**:

| App Server | User |
| ---------- | ------ |
| stapp01 | tony |
| stapp02 | steve |
| stapp03 | banner |

***

### High-Level Flow

1. Generate SSH key on **jump\_host** as user `thor`
2. Copy the **public key** to each app server user
3. Verify password-less SSH access

***

### Step-by-Step Solution

***

#### Step 1: Login to Jump Host as `thor`

```bash
ssh thor@jump_host.stratos.xfusioncorp.com
# password: mjolnir123
```

***

#### Step 2: Generate SSH Key (if not already present)

Run **as thor**:

```bash
ssh-keygen -t rsa -b 2048
```

* Press **Enter** for all prompts
* This creates:
  * Private key: `~/.ssh/id_rsa`
  * Public key: `~/.ssh/id_rsa.pub`

***

#### Step 3: Copy SSH Key to App Servers

** stapp01 (user: tony)**

```bash
ssh-copy-id tony@stapp01.stratos.xfusioncorp.com
# password: Ir0nM@n
```

** stapp02 (user: steve)**

```bash
ssh-copy-id steve@stapp02.stratos.xfusioncorp.com
# password: Am3ric@
```

** stapp03 (user: banner)**

```bash
ssh-copy-id banner@stapp03.stratos.xfusioncorp.com
# password: BigGr33n
```

- This automatically:

* Creates `~/.ssh` on target (if missing)
* Adds public key to `authorized_keys`
* Fixes permissions

***

#### Step 4: Verify Password-less Access

Run from jump host:

```bash
ssh tony@stapp01.stratos.xfusioncorp.com
ssh steve@stapp02.stratos.xfusioncorp.com
ssh banner@stapp03.stratos.xfusioncorp.com
```

- **You should NOT be prompted for a password**

***

### Expected Permissions (Auto-handled but good to know)

On app servers:

```
~/.ssh            → 700
~/.ssh/authorized_keys → 600
```

***

### Final Validation (Important for Exam/Lab)

* SSH works **without password**
* Access is via **correct sudo users**
* Keys originate from **thor on jump\_host**

***

### One-Line Summary (For Documentation)

> Configured SSH key-based authentication from `thor@jump_host` to all Nautilus app servers using their respective sudo users (`tony`, `steve`, `banner`) to enable password-less automation.

```
thor@jumphost ~$ ssh thor@jump_host.stratos.xfusioncorp.com
The authenticity of host 'jump_host.stratos.xfusioncorp.com (172.16.238.2)' can't be established.
ED25519 key fingerprint is SHA256:dUPR1TCj849vNVqC6xGhuVlMTnI8lrKb1fOfzt23ntE.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'jump_host.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
thor@jump_host.stratos.xfusioncorp.com's password: 
Last login: Tue Dec 23 15:33:39 2025
thor@jumphost ~$ ssh-keygen -t rsa -b 2048
Generating public/private rsa key pair.
Enter file in which to save the key (/home/thor/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/thor/.ssh/id_rsa
Your public key has been saved in /home/thor/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ijKm1JeOg1xD7vw6fdeVK2ax3r2AXC5TXLXx3SU5FKk thor@jumphost.stratos.xfusioncorp.com
The key's randomart image is:
+---[RSA 2048]----+
|             .+=o|
|              +.O|
|             . ++|
|   .        E .  |
|  o     S    +.  |
|  .+ ...  ..=o   |
|..O.+o.   .=+o.  |
|.= B+. . . *+.o  |
|.  o=o. . +... o.|
+----[SHA256]-----+
thor@jumphost ~$ 
thor@jumphost ~$ ssh-copy-id tony@stapp01.stratos.xfusioncorp.com
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"
The authenticity of host 'stapp01.stratos.xfusioncorp.com (172.17.0.5)' can't be established.
ED25519 key fingerprint is SHA256:zljSGnzueDf+eTqcd71EF0SDZ1iMQ99rBmxlIbZeFEQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
tony@stapp01.stratos.xfusioncorp.com's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'tony@stapp01.stratos.xfusioncorp.com'"
and check to make sure that only the key(s) you wanted were added.

thor@jumphost ~$ ssh-copy-id steve@stapp02.stratos.xfusioncorp.com
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"
The authenticity of host 'stapp02.stratos.xfusioncorp.com (172.17.0.7)' can't be established.
ED25519 key fingerprint is SHA256:Gq0PgzSUmqeT8xLkF++Ty9yczepunsi1537o1LtRrsg.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
steve@stapp02.stratos.xfusioncorp.com's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'steve@stapp02.stratos.xfusioncorp.com'"
and check to make sure that only the key(s) you wanted were added.

thor@jumphost ~$ ssh-copy-id banner@stapp03.stratos.xfusioncorp.com
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"
The authenticity of host 'stapp03.stratos.xfusioncorp.com (172.17.0.4)' can't be established.
ED25519 key fingerprint is SHA256:xWrvskbN+xn0ES9bBH5pnLmwGCiYULKuvqHf5mCpmRI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
banner@stapp03.stratos.xfusioncorp.com's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'banner@stapp03.stratos.xfusioncorp.com'"
and check to make sure that only the key(s) you wanted were added.

thor@jumphost ~$ ssh tony@stapp01.stratos.xfusioncorp.com
[tony@stapp01 ~]$ exit
logout
Connection to stapp01.stratos.xfusioncorp.com closed.
thor@jumphost ~$ ssh steve@stapp02.stratos.xfusioncorp.com
[steve@stapp02 ~]$ exit
logout
Connection to stapp02.stratos.xfusioncorp.com closed.
thor@jumphost ~$ ssh banner@stapp03.stratos.xfusioncorp.com
[banner@stapp03 ~]$ exit
logout
Connection to stapp03.stratos.xfusioncorp.com closed.
thor@jumphost ~$ 
```

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
