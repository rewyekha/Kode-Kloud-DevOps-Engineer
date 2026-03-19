# Day 13: IPtables Installation And Configuration
We have one of our websites up and running on our `Nautilus` infrastructure in `Stratos DC`. Our security team has raised a concern that right now Apache’s port i.e `5001` is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:

1\. Install `iptables` and all its dependencies on each app host.

2\. Block incoming port `5001` on all apps for everyone except for LBR host.

3\. Make sure the rules remain, even after system reboot.

```bash
thor@jumphost ~$ ssh tony@stapp01.stratos.xfusioncorp.com
The authenticity of host 'stapp01.stratos.xfusioncorp.com (172.17.0.8)' can't be established.
ED25519 key fingerprint is SHA256:PZ24SWKcJjsi1kKTal6hPM8nglWPew231Wbq0o39zio.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
tony@stapp01.stratos.xfusioncorp.com's password:
[tony@stapp01 ~]$ sudo yum install -y iptables-services

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
CentOS Stream 9 - BaseOS                                                                    43 kB/s | 6.7 kB     00:00
CentOS Stream 9 - BaseOS                                                                    10 MB/s | 8.8 MB     00:00
CentOS Stream 9 - AppStream                                                                 34 kB/s | 6.8 kB     00:00
CentOS Stream 9 - AppStream                                                                 11 MB/s |  26 MB     00:02
CentOS Stream 9 - Extras packages                                                           45 kB/s | 7.3 kB     00:00
CentOS Stream 9 - Extras packages                                                           45 kB/s |  20 kB     00:00
Docker CE Stable - x86_64                                                                   29 kB/s | 2.0 kB     00:00
Docker CE Stable - x86_64                                                                  292 kB/s |  65 kB     00:00
Extra Packages for Enterprise Linux 9 - x86_64                                             129 kB/s |  33 kB     00:00
Extra Packages for Enterprise Linux 9 - x86_64                                              34 MB/s |  20 MB     00:00
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                       5.1 kB/s | 993  B     00:00
Extra Packages for Enterprise Linux 9 - Next - x86_64                                       93 kB/s |  21 kB     00:00
Extra Packages for Enterprise Linux 9 - Next - x86_64                                      287 kB/s | 259 kB     00:00
Dependencies resolved.
===========================================================================================================================
 Package                            Architecture            Version                            Repository             Size
===========================================================================================================================
Installing:
 iptables-services                  noarch                  1.8.10-11.1.el9                    epel                   17 k

Transaction Summary
===========================================================================================================================
Install  1 Package

Total download size: 17 k
Installed size: 27 k
Downloading Packages:
iptables-services-1.8.10-11.1.el9.noarch.rpm                                                71 kB/s |  17 kB     00:00
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                       41 kB/s |  17 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                   1/1
  Installing       : iptables-services-1.8.10-11.1.el9.noarch                                                          1/1
  Running scriptlet: iptables-services-1.8.10-11.1.el9.noarch                                                          1/1
  Verifying        : iptables-services-1.8.10-11.1.el9.noarch                                                          1/1

Installed:
  iptables-services-1.8.10-11.1.el9.noarch

Complete!
[tony@stapp01 ~]$ sudo systemctl enable iptables
Created symlink /etc/systemd/system/multi-user.target.wants/iptables.service → /usr/lib/systemd/system/iptables.service.
[tony@stapp01 ~]$ sudo systemctl start iptables
[tony@stapp01 ~]$ sudo iptables -F
[tony@stapp01 ~]$ sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 5001 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -A INPUT -p tcp --dport 5001 -j DROP
[tony@stapp01 ~]$ sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
[tony@stapp01 ~]$ sudo service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[tony@stapp01 ~]$ sudo iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:5001
2    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:5001
3    ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
4    ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination
[tony@stapp01 ~]$ sudo reboot
You can not reboot this server. If you are done with your task, please click on the Finish button.
[tony@stapp01 ~]$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:5001
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:5001
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[tony@stapp01 ~]$
```

### Objective Recap
We must secure **Apache port 5001** on **all application servers**:

* **stapp01** – `172.16.238.10`
* **stapp02** – `172.16.238.11`
* **stapp03** – `172.16.238.12`

#### Security Rules Required
1. Install **iptables** and dependencies
2. **Block port 5001 for everyone**
3. **Allow port 5001 ONLY from Load Balancer (stlb01 – 172.16.238.14)**
4. Rules must **persist after reboot**

***

### Solution Overview (Logic)
```
Internet / Others    ---> port 5001 (BLOCKED)
Load Balancer        ---> port 5001 (ALLOWED)
```

We’ll:

* Install `iptables-services`
* Add **ALLOW rule first**
* Add **DENY rule next**
* Save rules permanently

***

### Perform the following steps on EACH App Host
 **stapp01, stapp02, stapp03**

***

### 1 Login to the App Server
Example for **stapp01**:

```bash
ssh tony@stapp01.stratos.xfusioncorp.com
```

(Use correct user/password for each server)

***

### 2 Install iptables & services
#### RHEL / CentOS / Rocky / Alma
```bash
sudo yum install -y iptables-services
```

***

### 3 Enable & Start iptables Service
```bash
sudo systemctl enable iptables
sudo systemctl start iptables
```

***

### 4 Flush Existing Rules (Safe Clean Start)
```bash
sudo iptables -F
```

***

### 5 Allow Port 5001 ONLY from Load Balancer
 **Note:**

```bash
sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 5001 -j ACCEPT
```

***

### 6 Block Port 5001 for Everyone Else
```bash
sudo iptables -A INPUT -p tcp --dport 5001 -j DROP
```

***

### 7 Allow Established Connections (Best Practice)
```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

***

### 8 Allow SSH (DO NOT LOCK YOURSELF OUT)
```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

***

### 9 Save Rules Permanently (Persistence After Reboot)
```bash
sudo service iptables save
```

 Rules saved to:

```
/etc/sysconfig/iptables
```

***

### Repeat Steps 1–9 on:
* `stapp02`
* `stapp03`

***

### Verification
#### Check rules
```bash
sudo iptables -L -n --line-numbers
```

Expected output (example):

```
ACCEPT tcp -- 172.16.238.14  0.0.0.0/0  tcp dpt:5001
DROP   tcp -- 0.0.0.0/0      0.0.0.0/0  tcp dpt:5001
```

***

### Reboot Test (Optional but Recommended)
```bash
sudo reboot
```

After reboot:

```bash
sudo iptables -L -n
```

 Rules should still exist → **Requirement #3 satisfied**

***

### Final Result
| Requirement               | Status |
| ------------------------- | ------ |
| iptables installed        |       |
| Port 5001 blocked for all |       |
| Only LBR allowed          |       |
| Persistent after reboot   |       |
