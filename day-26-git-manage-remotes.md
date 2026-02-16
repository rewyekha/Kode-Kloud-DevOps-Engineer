# Day 26: Git Manage Remotes

This document outlines the complete terminal execution performed on the **Stratos DC** Storage Server to:

* Add a new Git remote `dev_ecommerce`
* Commit a new file to the `master` branch
* Push changes to the new remote repository

***

### Repository Details

* **Local Repository:** `/usr/src/kodekloudrepos/ecommerce`
* **Existing Remote:** `/opt/ecommerce.git`
* **New Remote:** `/opt/xfusioncorp_ecommerce.git`
* **Server:** ststor01.stratos.xfusioncorp.com
* **User:** natasha

***

## Step 1: Connect to Storage Server

From jump host:

```bash
thor@jumphost ~$ ssh natasha@ststor01.stratos.xfusioncorp.com
```

Host authenticity confirmation:

```
The authenticity of host 'ststor01.stratos.xfusioncorp.com (172.16.238.15)' can't be established.
ED25519 key fingerprint is SHA256:OkOXDMlxVN2Sta+SKWuOrBNtdyQBWjEpfmOoobY4a3s.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
```

Login successful:

```
natasha@ststor01.stratos.xfusioncorp.com's password:
```

***

## Step 2: Navigate to Repository

```bash
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/ecommerce
```

Attempt to check branch:

```bash
[natasha@ststor01 ecommerce]$ git branch
```

Output:

```
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/ecommerce'
To add an exception for this directory, call:

        git config --global --add safe.directory /usr/src/kodekloudrepos/ecommerce
```

***

## Step 3: Switch to Root User

Because the repository is owned by root:

```bash
[natasha@ststor01 ecommerce]$ sudo -i
```

System message:

```
We trust you have received the usual lecture from the local System Administrator.
```

Navigate to repository:

```bash
[root@ststor01 ~]# cd /usr/src/kodekloudrepos/ecommerce
```

Verify branch:

```bash
[root@ststor01 ecommerce]# git branch
```

Output:

```
* master
```

***

## Step 4: Add New Remote

Add new remote `dev_ecommerce`:

```bash
[root@ststor01 ecommerce]# git remote add dev_ecommerce /opt/xfusioncorp_ecommerce.git
```

Verify remotes:

```bash
[root@ststor01 ecommerce]# git remote -v
```

Output:

```
dev_ecommerce   /opt/xfusioncorp_ecommerce.git (fetch)
dev_ecommerce   /opt/xfusioncorp_ecommerce.git (push)
origin          /opt/ecommerce.git (fetch)
origin          /opt/ecommerce.git (push)
```

***

## Step 5: Copy File into Repository

```bash
[root@ststor01 ecommerce]# cp /tmp/index.html .
```

Verify file:

```bash
[root@ststor01 ecommerce]# ls -l index.html
```

Output:

```
-rw-r--r-- 1 root root 120 Feb 16 16:09 index.html
```

***

## Step 6: Add and Commit Changes

```bash
[root@ststor01 ecommerce]# git add index.html
[root@ststor01 ecommerce]# git commit -m "Added index.html file to master branch"
```

Commit output:

```
[master 3ef814b] Added index.html file to master branch
 1 file changed, 10 insertions(+)
 create mode 100644 index.html
```

***

## Step 7: Push Master to New Remote

```bash
[root@ststor01 ecommerce]# git push dev_ecommerce master
```

Push output:

```
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 16 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 600 bytes | 600.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/xfusioncorp_ecommerce.git
 * [new branch]      master -> master
```

***

## Step 8: Verification

Verify remote branch:

```bash
[root@ststor01 ecommerce]# git ls-remote --heads dev_ecommerce
```

Output:

```
3ef814b31e65256723446c650dee6f8e32d0a273        refs/heads/master
```

***

## Final Outcome

✔ Added new remote `dev_ecommerce`\
✔ Copied `/tmp/index.html` into repository\
✔ Committed changes to `master`\
✔ Successfully pushed `master` branch to `/opt/xfusioncorp_ecommerce.git`\
✔ Verified remote branch exists

***

✅ **Task Completed Successfully**
