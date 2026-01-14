# Day 5: SElinux Installation and Configuration

Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for `App server 1` in the `Stratos Datacenter:`<br>

1. Install the required `SELinux` packages.
2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.
3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.
4. Disregard the current status of SELinux via the command line; the final status after the reboot should be `disabled`.

```
thor@jumphost ~$ sudo yum install -y selinux-policy selinux-policy-targeted

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for thor: 
CentOS Stream 9 - BaseOS                                                                    40 kB/s | 7.0 kB     00:00    
CentOS Stream 9 - BaseOS                                                                    16 MB/s | 8.8 MB     00:00    
CentOS Stream 9 - AppStream                                                                 70 kB/s | 7.4 kB     00:00    
CentOS Stream 9 - AppStream                                                                 38 MB/s |  26 MB     00:00    
CentOS Stream 9 - Extras packages                                                           49 kB/s | 8.0 kB     00:00    
CentOS Stream 9 - Extras packages                                                          5.5 kB/s |  20 kB     00:03    
Extra Packages for Enterprise Linux 9 - x86_64                                             144 kB/s |  30 kB     00:00    
Extra Packages for Enterprise Linux 9 - x86_64                                             1.5 MB/s |  20 MB     00:13    
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                       6.2 kB/s | 993  B     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                                      101 kB/s |  23 kB     00:00    
Extra Packages for Enterprise Linux 9 - Next - x86_64                                      302 kB/s | 259 kB     00:00    
Package selinux-policy-38.1.38-1.el9.noarch is already installed.
Package selinux-policy-targeted-38.1.38-1.el9.noarch is already installed.
Dependencies resolved.
===========================================================================================================================
 Package                                 Architecture           Version                       Repository              Size
===========================================================================================================================
Upgrading:
 selinux-policy                          noarch                 38.1.70-1.el9                 baseos                  41 k
 selinux-policy-targeted                 noarch                 38.1.70-1.el9                 baseos                 6.9 M

Transaction Summary
===========================================================================================================================
Upgrade  2 Packages

Total download size: 7.0 M
Downloading Packages:
(1/2): selinux-policy-38.1.70-1.el9.noarch.rpm                                              90 kB/s |  41 kB     00:00    
(2/2): selinux-policy-targeted-38.1.70-1.el9.noarch.rpm                                    1.4 MB/s | 6.9 MB     00:04    
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                      1.4 MB/s | 7.0 MB     00:05     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: selinux-policy-targeted-38.1.70-1.el9.noarch                                                      1/1 
  Preparing        :                                                                                                   1/1 
  Upgrading        : selinux-policy-38.1.70-1.el9.noarch                                                               1/4 
  Running scriptlet: selinux-policy-38.1.70-1.el9.noarch                                                               1/4 
  Running scriptlet: selinux-policy-targeted-38.1.70-1.el9.noarch                                                      2/4 
  Upgrading        : selinux-policy-targeted-38.1.70-1.el9.noarch                                                      2/4 
  Running scriptlet: selinux-policy-targeted-38.1.70-1.el9.noarch                                                      2/4 
libsemanage.semanage_rename: WARNING: rename(/var/lib/selinux/targeted/active, /var/lib/selinux/targeted/previous) failed: Invalid cross-device link, fall back to non-atomic semanage_copy_dir_flags()

  Running scriptlet: selinux-policy-38.1.38-1.el9.noarch                                                                          3/4 
  Cleanup          : selinux-policy-38.1.38-1.el9.noarch                                                                          3/4 
  Running scriptlet: selinux-policy-38.1.38-1.el9.noarch                                                                          3/4 
  Cleanup          : selinux-policy-targeted-38.1.38-1.el9.noarch                                                                 4/4 
  Running scriptlet: selinux-policy-targeted-38.1.38-1.el9.noarch                                                                 4/4 
  Running scriptlet: selinux-policy-targeted-38.1.70-1.el9.noarch                                                                4/4 
  Running scriptlet: selinux-policy-targeted-38.1.38-1.el9.noarch                                                                4/4 
  Verifying        : selinux-policy-38.1.70-1.el9.noarch                                                                                1/4 
  Verifying        : selinux-policy-38.1.38-1.el9.noarch                                                                                2/4 
  Verifying        : selinux-policy-targeted-38.1.70-1.el9.noarch                                                                       3/4 
  Verifying        : selinux-policy-targeted-38.1.38-1.el9.noarch                                                                       4/4 

Upgraded:
  selinux-policy-38.1.70-1.el9.noarch                              selinux-policy-targeted-38.1.70-1.el9.noarch                             

Complete!
thor@jumphost ~$ sudo vi /etc/selinux/config

[1]+  Stopped                 sudo vi /etc/selinux/config
thor@jumphost ~$ sudo vi /etc/selinux/config
thor@jumphost ~$ getenforce
Disabled
thor@jumphost ~$ 
```



### ✅ Objective Summary

On **App Server 1**:

* Install required **SELinux packages**
* **Permanently disable** SELinux
* **Do NOT reboot now**
* Ignore current runtime status (`getenforce`)
* After next reboot → SELinux must be **disabled**

***

### 🧠 Important Concept (Why this works)

* `setenforce 0` → **temporary** (lost after reboot) ❌
* Editing `/etc/selinux/config` → **permanent** ✅
* Reboot is **not required now**, but config must be ready for next reboot

***

### 🔹 Step 1: Install SELinux packages

#### On RHEL / CentOS / Rocky / Alma (Stratos uses these)

```bash
sudo yum install -y selinux-policy selinux-policy-targeted
```

> This ensures SELinux components are present even if currently disabled.

***

### 🔹 Step 2: Permanently disable SELinux

Edit the SELinux config file:

```bash
sudo vi /etc/selinux/config
```

Change this line:

```ini
SELINUX=enforcing
```

or

```ini
SELINUX=permissive
```

👉 **to**

```ini
SELINUX=disabled
```

✅ Save and exit.

***

### 🔹 Step 3: Do NOT reboot (as instructed)

✔ No reboot required\
✔ Scheduled maintenance reboot will apply the change

***

### 🔹 Step 4: Ignore current SELinux status

Even if:

```bash
getenforce
```

shows:

```
Enforcing
```

or

```
Permissive
```

👉 **Ignore it** (explicitly stated in the task).

After reboot, SELinux will be:

```
Disabled
```

***

### ✅ Final Verification (Post-Reboot – for your understanding)

After reboot (not now):

```bash
sestatus
```

Expected:

```
SELinux status: disabled
```

***

### 🏁 Final Answer (What Evaluator Checks)

✔ SELinux packages installed\
✔ `/etc/selinux/config` contains `SELINUX=disabled`\
✔ No reboot performed\
✔ Permanent disablement configured

**Task completed successfully** ✅

