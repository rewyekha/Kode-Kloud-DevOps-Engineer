# Day 14: Linux Process Troubleshooting

The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in `Stratos DC`.

Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don’t need to worry if Apache isn’t serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port `6100` on all app servers.



<pre><code><strong>thor@jumphost ~$ ssh tony@172.16.238.10
</strong>The authenticity of host '172.16.238.10 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:yHfHGhqXCBD4DoZFPnUf9hRwq0W6xDOIQFAWdsrsnEI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.10' (ED25519) to the list of known hosts.
tony@172.16.238.10's password: 
<strong>[tony@stapp01 ~]$ sudo systemctl status httpd
</strong>
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
Sorry, try again.
[sudo] password for tony: 
● httpd.service - The Apache HTTP Server
<strong>   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
</strong><strong>   Active: failed (Result: exit-code) since Thu 2026-01-08 23:46:15 UTC; 3min 10s ago
</strong><strong>     Docs: man:httpd(8)
</strong><strong>           man:apachectl(8)
</strong><strong>  Process: 675 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
</strong><strong>  Process: 674 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
</strong><strong> Main PID: 674 (code=exited, status=1/FAILURE)
</strong>
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: AH00558: httpd: Could not reliably determine the server'...sage
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: (98)Address already in use: AH00072: make_sock: could no...6100
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: no listening sockets available, shutting down
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: AH00015: Unable to open logs
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: main process exited, code=exited, status=...LURE
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com kill[675]: kill: cannot find process ""
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: control process exited, code=exited status=1
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to start The Apache HTTP Server.
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: Unit httpd.service entered failed state.
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service failed.
Hint: Some lines were ellipsized, use -l to show in full.
[tony@stapp01 ~]$ ssh steve@172.16.238.11
-bash: ssh: command not found
[tony@stapp01 ~]$ exit
logout
Connection to 172.16.238.10 closed.
<strong>thor@jumphost ~$ ssh steve@172.16.238.11
</strong>The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:+rxq/ty4ulgM8Mr+jngii5+6HKfICFlvnp8kQfV7BtM.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password: 
<strong>[steve@stapp02 ~]$ sudo systemctl status httpd
</strong>
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
<strong>     Active: active (running) since Thu 2026-01-08 23:46:16 UTC; 4min 39s ago
</strong>       Docs: man:httpd.service(8)
   Main PID: 1662 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 177 (limit: 411434)
     Memory: 19.4M
     CGroup: /docker/f84fbafd2f573bf8058ddc65ea47a9ee53c6fac1ced4f7e72ed41e835341bd8d/system.slice/httpd.service
             ├─1662 /usr/sbin/httpd -DFOREGROUND
             ├─1669 /usr/sbin/httpd -DFOREGROUND
             ├─1670 /usr/sbin/httpd -DFOREGROUND
             ├─1671 /usr/sbin/httpd -DFOREGROUND
             └─1672 /usr/sbin/httpd -DFOREGROUND

Jan 08 23:49:25 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:49:35 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:49:45 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:49:55 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:05 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:15 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:25 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:35 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:45 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>
Jan 08 23:50:55 stapp02.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1662 (READY=1>

<strong>[steve@stapp02 ~]$ ssh banner@172.16.238.12
</strong>The authenticity of host '172.16.238.12 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:Psl7giArvxAEyGOiTKeJAE4SnoxpaWs7jj9pifMUfgI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.12' (ED25519) to the list of known hosts.
banner@172.16.238.12's password: 
<strong>[banner@stapp03 ~]$ sudo systemctl status httpd
</strong>
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
<strong>     Active: active (running) since Thu 2026-01-08 23:46:16 UTC; 5min ago
</strong>       Docs: man:httpd.service(8)
   Main PID: 1669 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 177 (limit: 411434)
     Memory: 19.6M
     CGroup: /docker/a57ca128bd23bb4964f3e502998208f2718fb5edc49890eba177f5c906c66809/system.slice/httpd.service
             ├─1669 /usr/sbin/httpd -DFOREGROUND
             ├─1676 /usr/sbin/httpd -DFOREGROUND
             ├─1677 /usr/sbin/httpd -DFOREGROUND
             ├─1678 /usr/sbin/httpd -DFOREGROUND
             └─1679 /usr/sbin/httpd -DFOREGROUND

Jan 08 23:50:25 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:50:35 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:50:45 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:50:55 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:05 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:15 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:25 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:35 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:45 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>
Jan 08 23:51:55 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.service: Got notification message from PID 1669 (READY=1>

<strong>[banner@stapp03 ~]$ ssh tony@stapp01.stratos.xfusioncorp.com
</strong>The authenticity of host 'stapp01.stratos.xfusioncorp.com (172.17.0.5)' can't be established.
ED25519 key fingerprint is SHA256:yHfHGhqXCBD4DoZFPnUf9hRwq0W6xDOIQFAWdsrsnEI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
tony@stapp01.stratos.xfusioncorp.com's password: 
Last login: Thu Jan  8 23:49:03 2026 from jump_host.linux-process-v2_app_net
[tony@stapp01 ~]$ sudo systemctl status httpd
[sudo] password for tony: 
Sorry, try again.
[sudo] password for tony: 
<strong>● httpd.service - The Apache HTTP Server
</strong><strong>   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
</strong><strong>   Active: failed (Result: exit-code) since Thu 2026-01-08 23:46:15 UTC; 6min ago
</strong>     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 675 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
  Process: 674 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
 Main PID: 674 (code=exited, status=1/FAILURE)

Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: AH00558: httpd: Could not reliably determine the server'...sage
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: (98)Address already in use: AH00072: make_sock: could no...6100
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: no listening sockets available, shutting down
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com httpd[674]: AH00015: Unable to open logs
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: main process exited, code=exited, status=...LURE
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com kill[675]: kill: cannot find process ""
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service: control process exited, code=exited status=1
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to start The Apache HTTP Server.
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: Unit httpd.service entered failed state.
Jan 08 23:46:15 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.service failed.
Hint: Some lines were ellipsized, use -l to show in full.
<strong>[tony@stapp01 ~]$ sudo systemctl start httpd
</strong>Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
<strong>[tony@stapp01 ~]$ sudo systemctl enable httpd
</strong>Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[tony@stapp01 ~]$ sudo systemctl start httpd
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
<strong>[tony@stapp01 ~]$ sudo ss -tulnp | grep 6100
</strong><strong>tcp    LISTEN     0      10     127.0.0.1:6100                  *:*                   users:(("sendmail",pid=649,fd=4))
</strong><strong>[tony@stapp01 ~]$ sudo netstat -tulnp | grep 6100
</strong><strong>tcp        0      0 127.0.0.1:6100          0.0.0.0:*               LISTEN      649/sendmail: accep 
</strong>[tony@stapp01 ~]$ sudo systemctl stop sendmail
[tony@stapp01 ~]$ sudo systemctl disable sendmail
Removed symlink /etc/systemd/system/multi-user.target.wants/sendmail.service.
Removed symlink /etc/systemd/system/multi-user.target.wants/sm-client.service.
<strong>[tony@stapp01 ~]$ sudo ss -tulnp | grep 6100
</strong><strong>[tony@stapp01 ~]$ sudo systemctl start httpd
</strong><strong>[tony@stapp01 ~]$ sudo systemctl status httpd
</strong>● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
<strong>   Active: active (running) since Thu 2026-01-08 23:55:36 UTC; 8s ago
</strong>     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1276 (httpd)
   Status: "Processing requests..."
   CGroup: /docker/78b0367f09fd2ab9c91b3acec4c83243f024cfc7240703e62ce82aee7def5bee/system.slice/httpd.service
           ├─1276 /usr/sbin/httpd -DFOREGROUND
           ├─1277 /usr/sbin/httpd -DFOREGROUND
           ├─1278 /usr/sbin/httpd -DFOREGROUND
           ├─1279 /usr/sbin/httpd -DFOREGROUND
           ├─1280 /usr/sbin/httpd -DFOREGROUND
           └─1281 /usr/sbin/httpd -DFOREGROUND

Jan 08 23:55:36 stapp01.stratos.xfusioncorp.com systemd[1]: Starting The Apache HTTP Server...
Jan 08 23:55:36 stapp01.stratos.xfusioncorp.com httpd[1276]: AH00558: httpd: Could not reliably determine the server...sage
Jan 08 23:55:36 stapp01.stratos.xfusioncorp.com systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[tony@stapp01 ~]$ ^C
<strong>[tony@stapp01 ~]$ sudo ss -tulnp | grep httpd
</strong>tcp    LISTEN     0      511       *:6100                  *:*                   users:(("httpd",pid=1281,fd=3),("httpd",pid=1280,fd=3),("httpd",pid=1279,fd=3),("httpd",pid=1278,fd=3),("httpd",pid=1277,fd=3),("httpd",pid=1276,fd=3))
[tony@stapp01 ~]$ 
<strong>[tony@stapp01 ~]$ sudo systemctl enable httpd
</strong>[tony@stapp01 ~]$ 
[tony@stapp01 ~]$ sudo ss -tulnp | grep 6100
tcp    LISTEN     0      511       *:6100                  *:*                   users:(("httpd",pid=1281,fd=3),("httpd",pid=1280,fd=3),("httpd",pid=1279,fd=3),("httpd",pid=1278,fd=3),("httpd",pid=1277,fd=3),("httpd",pid=1276,fd=3))
[tony@stapp01 ~]$ 
</code></pre>

### 🎯 Task Summary

You must:

1. **Identify which app server has Apache down**
2. **Fix Apache**
3. **Ensure Apache is running on port `6100`**
4. **Apache must be running on ALL app servers**

App servers:

* `stapp01`
* `stapp02`
* `stapp03`

***

### 🔍 Step 1: Check Apache Status on All App Servers

Login to **each app server** and run:

```bash
sudo systemctl status httpd
```

#### Expected findings

* Two servers → `active (running)`
* **One server → inactive / failed / not running** ← ❌ faulty host

👉 **That server is the faulty app host**

***

### 🔧 Step 2: Start Apache on the Faulty Host

On the server where Apache is **not running**:

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

Recheck:

```bash
sudo systemctl status httpd
```

Apache should now show:

```
active (running)
```

***

### 🔁 Step 3: Ensure Apache Runs on Port 6100 (ALL App Servers)

This step is required **on all three app hosts**, even if Apache is already running.

***

#### 3.1 Edit Apache Port Configuration

```bash
sudo vi /etc/httpd/conf/httpd.conf
```

#### Find:

```apache
Listen 80
```

#### Replace with:

```apache
Listen 6100
```

***

#### 3.2 (If present) Update VirtualHost

Search for:

```apache
<VirtualHost *:80>
```

Change to:

```apache
<VirtualHost *:6100>
```

Save & exit (`:wq`).

***

### 🔄 Step 4: Restart Apache on ALL App Servers

```bash
sudo systemctl restart httpd
```

***

### ✅ Step 5: Verify Apache Is Running on Port 6100

Run on **each app server**:

```bash
sudo netstat -tulnp | grep httpd
```

or

```bash
sudo ss -tulnp | grep httpd
```

Expected output:

```
LISTEN 0 128 :::6100 :::* users:(("httpd",pid=XXXX))
```

***

### 🧪 Final Validation Checklist

| Check          | Command                      | Expected |
| -------------- | ---------------------------- | -------- |
| Apache running | `systemctl status httpd`     | active   |
| Correct port   | `ss -tulnp \| grep 6100`     | LISTEN   |
| Auto-start     | `systemctl is-enabled httpd` | enabled  |

***

### 🏁 Final Answer (What the task wants)

✔ Faulty app host identified and fixed\
✔ Apache running on **all app servers**\
✔ Apache listening on **port 6100**\
✔ No need for web content (service-only task)

👉 **Task is COMPLETE and ready for submission** ✅
