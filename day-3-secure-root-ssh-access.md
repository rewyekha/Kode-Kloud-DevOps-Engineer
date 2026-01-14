# Day 3: Secure Root SSH Access

Following security audits, the `xFusionCorp Industries` security team has rolled out new protocols, including the restriction of direct root SSH login.\
[Your task is to disable direct SSH root login on all app servers within the `Stratos Datacenter`.](#user-content-fn-1)[^1]

To complete this task in the **Project Nautilus** environment on KodeKloud Engineer, disable direct SSH root login on the three application servers: **stapp01**, **stapp02**, and **stapp03**.

Access these servers via the **jump host** (using user **thor** and password **mjolnir123**), as direct access isn't available.

#### Steps to Follow

1.  From your local terminal or the KodeKloud lab terminal, SSH into each app server using the provided credentials:

    *   For **stapp01**:

        ```
        ssh tony@172.16.238.10
        ```

        (Password: `Ir0nM@n`)
    *   For **stapp02**:

        ```
        ssh steve@172.16.238.11
        ```

        (Password: `Am3ric@`)
    *   For **stapp03**:

        ```
        ssh banner@172.16.238.12
        ```

        (Password: `BigGr33n`)

    These users (**tony**, **steve**, **banner**) have sudo privileges.
2.  On **each** app server, gain root access (since editing the SSH config requires elevated privileges):

    ```
    sudo su -
    ```

    (Or use `sudo` for individual commands.)
3.  Edit the SSH server configuration file:

    ```
    vi /etc/ssh/sshd_config
    ```

    (Or use `nano` if preferred.)

    * Find the line with `PermitRootLogin` (it might be commented with `#` or set to `yes/prohibit-password`).
    * **Uncomment** it if needed (remove the `#`).
    *   Set it exactly to:

        ```
        PermitRootLogin no
        ```

    Save and exit (`:wq` in vi).

    **Important**: The file is `/etc/ssh/sshd_config` (server config), **not** `ssh_config` (client config). Many task failures occur due to editing the wrong file.
4.  Restart the SSH service to apply changes:

    ```
    systemctl restart sshd
    ```

    (Or `service sshd restart` if systemctl isn't available.)
5. Repeat steps 2–4 on **all three** servers.
6. (Optional) Verify on each server:
   * From the jump host, try `ssh root@<app-server-ip>` — it should fail with "Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password)".
   * Your existing session as the sudo user remains active.

After completing this on all three app servers, submit the task in the KodeKloud interface for validation.

This enhances security by preventing brute-force attacks directly targeting the root account, forcing logins via non-root users with sudo escalation.



```
thor@jumphost ~$ ssh tony@172.16.238.10
The authenticity of host '172.16.238.10 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:8JDJ6eEFGZKfMJntPYyYB8R+A4ubnU16ZhH0nm/q9XQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.10' (ED25519) to the list of known hosts.
tony@172.16.238.10's password: 
Permission denied, please try again.
tony@172.16.238.10's password: 
Last failed login: Fri Dec 19 16:54:13 UTC 2025 from 172.16.238.3 on ssh:notty
There was 1 failed login attempt since the last successful login.
[tony@stapp01 ~]$ sudo su -

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony: 
[root@stapp01 ~]# vi /etc/ssh/sshd_config
[root@stapp01 ~]# nano /etc/ssh/sshd_config
-bash: nano: command not found
[root@stapp01 ~]# vi /etc/ssh/sshd_config
[root@stapp01 ~]# sudo systemctl restart sshd
[root@stapp01 ~]# exit
logout
[tony@stapp01 ~]$ ssh steve@172.16.238.11
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:eLBrx5DJPVKDAtMupvUy7xNMbit/5ytfKQ8jVL3jC6M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password: 
[steve@stapp02 ~]$ sudo su -

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for steve: 
[root@stapp02 ~]# sudo su -
Last login: Fri Dec 19 17:09:54 UTC 2025 on pts/0
[root@stapp02 ~]# vi /etc/ssh/sshd_config
[root@stapp02 ~]# exit
logout
[root@stapp02 ~]# exit
logout
[steve@stapp02 ~]$ ssh banner@172.16.238.12
The authenticity of host '172.16.238.12 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:cw5HFE27hPn221Aec4smuZkgXni1ZwzkGo1i0daAVm0.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.12' (ED25519) to the list of known hosts.
banner@172.16.238.12's password: 
[banner@stapp03 ~]$ sudo su -

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
[root@stapp03 ~]# vi /etc/ssh/sshd_config
[root@stapp03 ~]# systemctl restart sshd
[root@stapp03 ~]# ssh steve@172.16.238.11
The authenticity of host '172.16.238.11 (172.16.238.11)' can't be established.
ED25519 key fingerprint is SHA256:eLBrx5DJPVKDAtMupvUy7xNMbit/5ytfKQ8jVL3jC6M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.11' (ED25519) to the list of known hosts.
steve@172.16.238.11's password: 
Last login: Fri Dec 19 17:09:30 2025 from 172.16.238.10
[steve@stapp02 ~]$ systemctl restart sshd
Failed to restart sshd.service: Access denied
See system logs and 'systemctl status sshd.service' for details.
[steve@stapp02 ~]$ sudo systemctl restart sshd
[sudo] password for steve: 
[steve@stapp02 ~]$ 
```

[^1]: 
