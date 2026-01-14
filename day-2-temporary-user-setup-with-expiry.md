# Day 2: Temporary User Setup with Expiry

As part of the temporary assignment to the `Nautilus` project, a developer named `kirsty` requires access for a limited duration. To ensure smooth access management, a temporary user account with an expiry date is needed. Here's what you need to do:\
Create a user named `kirsty` on `App Server 2` in Stratos Datacenter. Set the expiry date to `2024-03-28`, ensuring the user is created in lowercase as per standard protocol.

> Note: You can find the infrastructure details by clicking on the **Details of all Users and Servers** button on the top-right section of the page.



***

### 1. SSH into App Server 2

From the jump host (or wherever access is provided), connect to **stapp02**:

```bash
ssh steve@stapp02.stratos.xfusioncorp.com
```

Password:

```
Am3ric@
```

***

### 2. Create the User with Expiry Date

Create the user **kirsty** (lowercase) and set the expiry date to **2024-03-28**.

```bash
sudo useradd -e 2024-03-28 kirsty
```

> `-e` specifies the account expiration date in `YYYY-MM-DD` format.

***

### 3. (Optional but Recommended) Verify the Expiry Date

Confirm that the expiry date is set correctly:

```bash
sudo chage -l kirsty
```

You should see:

```
Account expires : Mar 28, 2024
```

***

### ✅ Task Completed

* User **kirsty** created on **App Server 2**
* Username in lowercase ✔
* Expiry date set to **2024-03-28** ✔



```
thor@jumphost ~$ ssh steve@stapp02.stratos.xfusioncorp.com
The authenticity of host 'stapp02.stratos.xfusioncorp.com (172.17.0.8)' can't be established.
ED25519 key fingerprint is SHA256:irzRg4c7YYLJkyAOcpfXBBY3Q7/PkP8etQgLhAqBFko.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp02.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
steve@stapp02.stratos.xfusioncorp.com's password: 
[steve@stapp02 ~]$ sudo useradd -e 2024-03-28 kirsty

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[steve@stapp02 ~]$ sudo chage -l kirsty
Last password change                                    : Dec 17, 2025
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Mar 28, 2024
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
[steve@stapp02 ~]$ 
```

