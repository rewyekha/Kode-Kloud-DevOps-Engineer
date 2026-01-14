# Day 4: Script Execution Permissions

In a bid to automate backup processes, the `xFusionCorp Industries` sysadmin team has developed a new bash script named `xfusioncorp.sh`. While the script has been distributed to all necessary servers, it lacks executable permissions on `App Server 3` within the Stratos Datacenter.<br>

**`Your task is to grant executable permissions to the /tmp/xfusioncorp.sh script on App Server 3. Additionally, ensure that all users have the capability to execute it.`**

To complete this task on **App Server 3** (stapp03), you need to grant executable permissions to the script `/tmp/xfusioncorp.sh` so that **all users** on the server can execute it.

#### Steps to Follow

1.  SSH into **App Server 3**:

    ```bash
    ssh banner@172.16.238.12
    ```

    Password: `BigGr33n`
2.  Once logged in, check the current permissions of the script:

    ```bash
    ls -l /tmp/xfusioncorp.sh
    ```

    (You'll likely see something like `-rw-r--r--` or similar, without the `x`).
3.  Add **executable permissions for everyone** (owner, group, and others):

    ```bash
    sudo chmod +x /tmp/xfusioncorp.sh
    ```

    OR explicitly set the permissions so all users can execute:

    ```bash
    sudo chmod 755 /tmp/xfusioncorp.sh
    ```

    * `7` = read + write + execute for owner
    * `5` = read + execute for group and others

    Both commands achieve the same result for this task.
4.  Verify the permissions have been updated:

    ```bash
    ls -l /tmp/xfusioncorp.sh
    ```

    You should now see something like:

    ```
    -rwxr-xr-x 1 root root ... /tmp/xfusioncorp.sh
    ```

    The important part is the `x` in owner, group, and others sections (`r-x` for group and others).
5.  (Optional) Test execution (you don't need to run the script, just confirm permissions are correct):

    ```bash
    /tmp/xfusioncorp.sh
    ```

    If it runs or shows script output/errors, permissions are set correctly.

#### Summary

* File: `/tmp/xfusioncorp.sh`
* Server: **App Server 3** (stapp03)
* Required command: `sudo chmod +x /tmp/xfusioncorp.sh` (or `sudo chmod 755 /tmp/xfusioncorp.sh`)

That's all! Once done, you can submit the task in KodeKloud for validation.

```
thor@jumphost ~$ ssh banner@172.16.238.12
The authenticity of host '172.16.238.12 (172.16.238.12)' can't be established.
ED25519 key fingerprint is SHA256:98jXA601kc7Krit8oDRPilBJGFGYlnJy3DO5vfdFW4M.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.238.12' (ED25519) to the list of known hosts.
banner@172.16.238.12's password: 
[banner@stapp03 ~]$ ls -l /tmp/xfusioncorp.sh
---------- 1 root root 40 Dec 19 17:40 /tmp/xfusioncorp.sh
[banner@stapp03 ~]$ sudo chmod +x /tmp/xfusioncorp.sh

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
[banner@stapp03 ~]$ ls -l /tmp/xfusioncorp.sh
---x--x--x 1 root root 40 Dec 19 17:40 /tmp/xfusioncorp.sh
[banner@stapp03 ~]$ sudo chmod 755 /tmp/xfusioncorp.sh
[banner@stapp03 ~]$ ls -l /tmp/xfusioncorp.sh
-rwxr-xr-x 1 root root 40 Dec 19 17:40 /tmp/xfusioncorp.sh
[banner@stapp03 ~]$ 

```
