# File Permission Correction

After conducting a security audit within the `Stratos DC`, the Nautilus security team discovered misconfigured permissions on critical files. To address this, corrective actions are being taken by the production support team. Specifically, the file named `/etc/hostname` on `Nautilus App 3` server requires adjustments to its Access Control Lists (ACLs) as follows:

1\. The file's user owner and group owner should be set to `root`.\
\
2\. `Others` should possess `read only` permissions on the file.\
\
3\. User `siva` must not have any permissions on the file.\
\
4\. User `ryan` should be granted `read only` permission on the file.



***

## Fix ACL Permissions on `/etc/hostname` (Nautilus App 3)

### Problem Statement

After conducting a security audit within the Stratos DC, the Nautilus security team discovered misconfigured permissions on critical files. To address this, corrective actions are required on the Nautilus App 3 server.

Update the Access Control Lists (ACLs) for the file `/etc/hostname` with the following requirements:

1. Set the user owner and group owner of the file to `root`.
2. Ensure others have read-only permissions.
3. User `siva` must not have any permissions.
4. User `ryan` should have read-only permissions.

***

### Infrastructure Details

| Server Name          | Hostname  | User   | Purpose                      |
| -------------------- | --------- | ------ | ---------------------------- |
| Application Server 3 | stapp03   | banner | Hosts Nautilus Application 3 |
| Jump Host            | jump-host | thor   | Provides secure access       |

***

### Solution Steps

#### Step 1: Connect to Nautilus App 3 Server

```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.29.209)' can't be established.
ED25519 key fingerprint is SHA256:jbExwZy0JkOW0yieTOTBwGdQkLe4T9+jvIe/Kz2N0sc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password:
```

***

#### Step 2: Switch to Root User

```bash
[banner@stapp03 ~]$ sudo -i

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
```

***

#### Step 3: Set Ownership to Root

```bash
[root@stapp03 ~]# chown root:root /etc/hostname
```

***

#### Step 4: Set Base File Permissions

```bash
[root@stapp03 ~]# chmod 644 /etc/hostname
```

***

#### Step 5: Update ACL Entries

**Remove all permissions for user `siva`**

```bash
[root@stapp03 ~]# setfacl -m u:siva:0 /etc/hostname
```

**Grant read-only permission to user `ryan`**

```bash
[root@stapp03 ~]# setfacl -m u:ryan:r-- /etc/hostname
```

***

#### Step 6: Verify ACL Configuration

```bash
[root@stapp03 ~]# getfacl /etc/hostname
getfacl: Removing leading '/' from absolute path names
# file: etc/hostname
# owner: root
# group: root
user::rw-
user:siva:---
user:ryan:r--
group::r--
mask::r--
other::r--
```

***

### Final Verification

| Requirement                         | Status    |
| ----------------------------------- | --------- |
| Owner is root                       | Completed |
| Group is root                       | Completed |
| Others have read-only               | Completed |
| User siva has no permissions        | Completed |
| User ryan has read-only permissions | Completed |

***

### Conclusion

The ACL and ownership of `/etc/hostname` on the Nautilus App 3 server have been successfully updated according to the security requirements. The configuration has been verified using `getfacl`.

```bash
thor@jump-host ~$ ssh banner@stapp03
The authenticity of host 'stapp03 (10.244.29.209)' can't be established.
ED25519 key fingerprint is SHA256:jbExwZy0JkOW0yieTOTBwGdQkLe4T9+jvIe/Kz2N0sc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp03' (ED25519) to the list of known hosts.
banner@stapp03's password: 
[banner@stapp03 ~]$ sudo -i

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner: 
[root@stapp03 ~]# chown root:root /etc/hostname
[root@stapp03 ~]# chmod 644 /etc/hostname
[root@stapp03 ~]# setfacl -m u:siva:0 /etc/hostname
[root@stapp03 ~]# setfacl -m u:ryan:r-- /etc/hostname
[root@stapp03 ~]# getfacl /etc/hostname
getfacl: Removing leading '/' from absolute path names
# file: etc/hostname
# owner: root
# group: root
user::rw-
user:siva:---
user:ryan:r--
group::r--
mask::r--
other::r--

[root@stapp03 ~]# 
```

