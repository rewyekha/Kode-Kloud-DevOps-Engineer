# Day 31: Git Stash

The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/ecommerce` present on `Storage server` in `Stratos DC`. One of the developers stashed some in-progress changes in this repository, but now they want to restore some of the stashed changes. Find below more details to accomplish this task:

Look for the stashed changes under `/usr/src/kodekloudrepos/ecommerce` git repository, and restore the stash with `stash@{1}` identifier. Further, commit and push your changes to the origin.



***

## Restore Git Stash and Push Changes

### 📌 Task Objective

Restore the stashed changes with identifier:

```
stash@{1}
```

From the Git repository located at:

```
/usr/src/kodekloudrepos/ecommerce
```

Then commit and push the changes to the remote origin.

***

## 🖥 Infrastructure Details

* **Server:** ststor01 (Storage Server)
* **User:** natasha
* **Repository Path:** `/usr/src/kodekloudrepos/ecommerce`
* **Branch:** master

***

## 🚀 Terminal Solution

### Step 1: SSH into Storage Server

```bash
ssh natasha@ststor01.stratos.xfusioncorp.com
```

Enter password when prompted.

***

### Step 2: Switch to Root User

```bash
sudo -i
```

Enter password again when prompted.

***

### Step 3: Navigate to Repository

```bash
cd /usr/src/kodekloudrepos/ecommerce
```

Verify repository status:

```bash
git status
```

Expected output:

```
On branch master
Your branch is up to date with 'origin/master'.
nothing to commit, working tree clean
```

***

### Step 4: Check Available Stashes

```bash
git stash list
```

Example output:

```
stash@{0}: WIP on master: 26a978a initial commit
stash@{1}: WIP on master: 26a978a initial commit
```

***

### Step 5: Apply Required Stash

```bash
git stash apply stash@{1}
```

Verify changes:

```bash
git status
```

Expected output:

```
Changes to be committed:
    new file: welcome.txt
```

***

### Step 6: Add Changes

```bash
git add .
```

***

### Step 7: Commit Changes

```bash
git commit -m "Restored changes from stash@{1}"
```

Example output:

```
1 file changed, 1 insertion(+)
create mode 100644 welcome.txt
```

***

### Step 8: Push to Remote Repository

```bash
git push origin master
```

Expected output:

```
master -> master
```

***

## ✅ Final Command Summary

```bash
ssh natasha@ststor01.stratos.xfusioncorp.com
sudo -i
cd /usr/src/kodekloudrepos/ecommerce
git stash list
git stash apply stash@{1}
git add .
git commit -m "Restored changes from stash@{1}"
git push origin master
```

***

## 🎯 Result

* Stash `stash@{1}` successfully restored
* Changes committed to `master`
* Updates pushed to remote origin

✔ Task Completed Successfully

***

