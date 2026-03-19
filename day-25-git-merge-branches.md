# Day 25: Git Merge Branches
The Nautilus application development team has been working on a project repository `/opt/official.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:

Create a new branch `datacenter` in `/usr/src/kodekloudrepos/official` repo from `master` and copy the `/tmp/index.html` file (present on `storage server` itself) into the repo. Further, `add/commit` this file in the new branch and merge back that branch into `master` branch. Finally, push the changes to the origin for both of the branches.

## GitBook Documentation
### Create `datacenter` Branch and Merge into `master`
This document describes the complete procedure performed on the **Stratos DC** storage server to:

* Create a new branch `datacenter` from `master`
* Copy `/tmp/index.html` into the repository
* Commit the file in the new branch
* Merge `datacenter` into `master`
* Push both branches to remote repository `/opt/official.git`

***

### Infrastructure Details
* **Jump Host:** jump\_host.stratos.xfusioncorp.com
* **Storage Server:** ststor01.stratos.xfusioncorp.com
* **Git Repository (local clone):** `/usr/src/kodekloudrepos/official`
* **Remote Repository:** `/opt/official.git`

***

## Step 1: Connect to Storage Server
Login from jump host to storage server:

```bash
thor@jumphost ~$ ssh natasha@ststor01.stratos.xfusioncorp.com
```

Accept the host authenticity prompt:

```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Enter password when prompted.

***

## Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/official
```

Attempting to check branches results in a dubious ownership warning:

```bash
git branch
```

Output:

```
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/official'
```

***

## Step 3: Switch to Root User
Since the repository is owned by root, switch user:

```bash
sudo -i
```

Navigate to repository:

```bash
cd /usr/src/kodekloudrepos/official
```

Verify current branch:

```bash
git branch
```

Output:

```
* master
```

***

## Step 4: Create New Branch `datacenter`
```bash
git checkout -b datacenter
```

Output:

```
Switched to a new branch 'datacenter'
```

***

## Step 5: Copy File into Repository
Copy the file from `/tmp`:

```bash
cp /tmp/index.html .
```

Verify file:

```bash
ls -l index.html
```

***

## Step 6: Add and Commit File
```bash
git add index.html
git commit -m "Added index.html file to datacenter branch"
```

Commit output:

```
[datacenter 6ee6280] Added index.html file to datacenter branch
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

***

## Step 7: Merge `datacenter` into `master`
Switch back to master:

```bash
git checkout master
```

Merge branch:

```bash
git merge datacenter
```

Output:

```
Updating 7ab2e85..6ee6280
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
```

***

## Step 8: Push Changes to Remote
Push `datacenter` branch:

```bash
git push origin datacenter
```

Push `master` branch:

```bash
git push origin master
```

Remote repository updated:

```
To /opt/official.git
 * [new branch]      datacenter -> datacenter
```

***

## Step 9: Verification
### Verify Local Branches
```bash
git branch -a
```

Output:

```
  datacenter
* master
  remotes/origin/datacenter
  remotes/origin/master
```

### Verify Remote Heads
```bash
git ls-remote --heads origin
```

Output:

```
6ee62802589381cb514b4487bb85abf44d75b40d        refs/heads/datacenter
6ee62802589381cb514b4487bb85abf44d75b40d        refs/heads/master
```

***

## Final Outcome
 Created new branch `datacenter`
 Copied `/tmp/index.html` into repository
 Committed changes in `datacenter`
 Merged `datacenter` into `master` (fast-forward merge)
 Pushed both branches to `/opt/official.git`
 Verified local and remote branches successfully

***

**Task Completed Successfully.**
