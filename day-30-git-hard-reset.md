# Day 30: Git hard reset
***

## Day 30: Git Hard Reset
***

### Question
The Nautilus application development team was working on a git repository:

```
/usr/src/kodekloudrepos/blog
```

This repository is present on the **Storage server** in Stratos DC.

This was just a test repository and one of the developers pushed several test commits. Now they want to clean the repository along with the commit history/work tree and point back the `HEAD` and the branch to the commit with message:

```
add data.txt file
```

#### Requirements:
* Reset the git commit history so that there are only **two commits**:
  * `initial commit`
  * `add data.txt file`
* Remove all later commits
* Push the changes to the remote repository

***

## Server Details
* Server: `ststor01`
* User: `natasha`
* Repository Path: `/usr/src/kodekloudrepos/blog`

***

## Solution
***

### Step 1: Login to Storage Server
From jump host:

```bash
thor@jumphost ~$ ssh natasha@ststor01
```

Output:

```bash
The authenticity of host 'ststor01 (172.16.238.15)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

***

### Step 2: Navigate to Repository
```bash
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/blog
```

Git shows dubious ownership:

```bash
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/blog'
```

Switch to root:

```bash
[natasha@ststor01 blog]$ sudo -i
```

Then:

```bash
[root@ststor01 ~]# cd /usr/src/kodekloudrepos/blog
```

Check repository status:

```bash
[root@ststor01 blog]# git status
```

Output:

```bash
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

***

### Step 3: Check Commit History
```bash
[root@ststor01 blog]# git log --oneline
```

Output:

```bash
316af1e (HEAD -> master, origin/master) Test Commit10
2d8c58a Test Commit9
304abfb Test Commit8
7ab86d0 Test Commit7
e63fd2f Test Commit6
5a73279 Test Commit5
5062f00 Test Commit4
4fca728 Test Commit3
cf15c0b Test Commit2
3f30fa3 Test Commit1
a2b2d0c add data.txt file
97144b5 initial commit
```

We need to reset to:

```
a2b2d0c add data.txt file
```

***

### Step 4: Hard Reset to Required Commit
```bash
[root@ststor01 blog]# git reset --hard a2b2d0c
```

Output:

```bash
HEAD is now at a2b2d0c add data.txt file
```

This removes all commits after `add data.txt file`.

***

### Step 5: Force Push to Remote
Since history was rewritten, a force push is required:

```bash
[root@ststor01 blog]# git push origin master --force
```

Output:

```bash
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/blog.git
 + 316af1e...a2b2d0c master -> master (forced update)
```

***

### Step 6: Verify Final Commit History
```bash
[root@ststor01 blog]# git log --oneline
```

Final Output:

```bash
a2b2d0c (HEAD -> master, origin/master) add data.txt file
97144b5 initial commit
```

***

## Final Result
 Only two commits remain
 `HEAD` points to `add data.txt file`
 All test commits removed
 Remote repository updated
 Working tree clean

***

## Final Command Summary
```bash
ssh natasha@ststor01
cd /usr/src/kodekloudrepos/blog
sudo -i
cd /usr/src/kodekloudrepos/blog
git log --oneline
git reset --hard a2b2d0c
git push origin master --force
git log --oneline
```

***

 **Task Completed Successfully**
