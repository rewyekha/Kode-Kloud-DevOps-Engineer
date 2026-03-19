# Day 12: Linux Network Services
Our monitoring tool has reported an issue in `Stratos Datacenter`. One of our app servers has an issue, as its Apache service is not reachable on port `3002` (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.

Use tools like `telnet`, `netstat`, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.

Once fixed, you can test the same using command `curl http://stapp01:3002` command from jump host.

`Note:` Please do not try to alter the existing `index.html` code, as it will lead to task failure.

###

> // connecting or not
> telnet stapp01 5003
> telnet stapp02 5003
> telnet stapp03 5003
>
> // app1 issue
> ssh tony@stapp01
> sudo su
> systemctl status httpd
> netstat -lntp | grep 5003
> kill 444
> netstat -lntp | grep 5003
>
> systemctl restart httpd
> systemctl status httpd
>
> telnet stapp01 5003
> // its due to the firewall
>
> iptables -L -n
>
> // how to open firewall port in iptables of linux
> sudo iptables -A INPUT -p tcp --dport 5003 -j ACCEPT
>
> // how to open firewall port sequence in iptables of linux
>
> sudo iptables -D INPUT -p tcp --dport 5004 -j ACCEPT
> sudo iptables -I INPUT 1 -p tcp --dport 5003 -j ACCEPT
> iptables -L -n

***

### STEP-BY-STEP (YouTube-style)
***

### 1 From jump host – check connectivity
```bash
ssh thor@jump_host
```

Test port:

```bash
telnet stapp01 3002
```

If it fails → problem confirmed

***

### 2 SSH into App Server 1
```bash
ssh tony@stapp01
sudo su
```

***

### 3 Check Apache service
```bash
systemctl status httpd
```

#### If Apache is not running:
```bash
systemctl start httpd
```

If it **fails**, continue below.

***

### 4 Check what is using port 3002
```bash
netstat -lntp | grep 3002
```

#### Possible outcomes:
**Case A: Some process is already using 3002 (like in YouTube video)**

Example:

```
tcp  0  0 127.0.0.1:3002  LISTEN  456/someprocess
```

Kill that PID (ONLY the one using 3002):

```bash
kill 456
```

Recheck:

```bash
netstat -lntp | grep 3002
```

***

### 5 Restart Apache
```bash
systemctl restart httpd
systemctl status httpd
```

Confirm Apache is listening:

```bash
netstat -lntp | grep 3002
```

You should see:

```
0.0.0.0:3002  LISTEN  httpd
```

***

### 6 Test locally on stapp01
```bash
curl http://localhost:3002
```

If this works → Apache is fine.

***

### 7 Check firewall
```bash
iptables -L -n
```

If port **3002 is NOT allowed**, open it (same style as YouTube):

```bash
iptables -I INPUT 1 -p tcp --dport 3002 -j ACCEPT
```

Verify:

```bash
iptables -L -n
```

Only port 3002 opened → **no security compromise**

***

### 8 Final test from jump host (THIS IS WHAT GRADER CHECKS)
Exit back:

```bash
exit
```

Test:

```bash
curl http://stapp01:3002
```

If this works → **TASK PASSES**

***

### Summary (YouTube Logic Applied Correctly)
| Step          | What you did            |
| ------------- | ----------------------- |
| telnet        | Verified issue          |
| netstat       | Found port usage        |
| kill PID      | Freed port              |
| restart httpd | Fixed Apache            |
| iptables      | Allowed port            |
| curl          | Verified from jump host |

***

### Very Important Notes
* Do **NOT** edit `index.html`
* Do **NOT** change Apache config unless required
* Only fix **service + port + firewall**

***

If you want, paste **only this output** and I’ll tell you the **exact next command**:

```bash
netstat -lntp | grep 3002
systemctl status httpd
```

```bash
[root@stapp01 tony]# history
    1  systemctl status httpd
    2  systemctl start httpd
    3  netstat -lntp | grep 8087
    4  systemctl stop sendmail
    5  systemctl disable sendmail
    6  netstat -lntp | grep 8087
    7  systemctl start httpd
    8  systemctl enable httpd
    9  systemctl status httpd
   10  netstat -lntp | grep 8087
   11  iptables -L -n
   12  curl http://localhost:8087
   13  sudo history
   14  history
   15  history
[root@stapp01 tony]#
```

```bash
[tony@stapp01 ~]$ iptables -L -n
iptables v1.8.4 (nf_tables): Could not fetch rule set generation id: Permission denied (you must be root)

[tony@stapp01 ~]$ sudo iptables -L -n
[sudo] password for tony:
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3002
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
# Warning: iptables-legacy tables present, use iptables-legacy to see them
[tony@stapp01 ~]$
```
