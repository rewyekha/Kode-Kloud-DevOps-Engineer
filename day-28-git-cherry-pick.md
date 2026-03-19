# Day 28: Git Cherry Pick
The Nautilus application development team has been working on a project repository /opt/ecommerce.git. This repo is cloned at /usr/src/kodekloudrepos on storage server in Stratos DC. They recently shared the following requirements with the DevOps team:

There are two branches in this repository, master and feature. One of the developers is working on the feature branch and their work is still in progress, however they want to merge one of the commits from the feature branch to the master branch, the message for the commit that needs to be merged into master is Update info.txt. Accomplish this task for them, also remember to push your changes eventually.

## Cherry-Pick a Specific Commit from Feature to Master
### Scenario
The Nautilus development team maintains a Git repository:

```
/opt/ecommerce.git
```

The repository is cloned at:

```
/usr/src/kodekloudrepos/ecommerce
```

There are two branches:

* `master`
* `feature`

A developer requested to merge **only one specific commit** from `feature` into `master`.

#### Required Commit
```
Update info.txt
```

***

## Step 1: Login to Storage Server
From jump host:

```bash
thor@jumphost ~$ ssh natasha@ststor01
```

Output:

```
The authenticity of host 'ststor01 (172.16.238.15)' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

***

## Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/ecommerce
```

If you see a dubious ownership warning:

```
fatal: detected dubious ownership in repository
```

Switch to root:

```bash
sudo -i
cd /usr/src/kodekloudrepos/ecommerce
```

***

## Step 3: Verify Available Branches
```bash
git branch
```

Output:

```
* feature
  master
```

***

## Step 4: Identify the Required Commit
Switch to `feature` branch:

```bash
git checkout feature
```

Check commit history:

```bash
git log --oneline
```

Output:

```
beb11d8 Update welcome.txt
2bb682f Update info.txt
418e307 Add welcome.txt
b43cc6a initial commit
```

 Required commit hash:

```
2bb682f
```

***

## Step 5: Switch to Master Branch
```bash
git checkout master
```

Output:

```
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

Pull latest changes:

```bash
git pull origin master
```

Output:

```
Already up to date.
```

***

## Step 6: Cherry-Pick Required Commit
```bash
git cherry-pick 2bb682f
```

Output:

```
[master 2f7c705] Update info.txt
 Date: Wed Feb 18 04:17:47 2026 +0000
 1 file changed, 1 insertion(+), 1 deletion(-)
```

***

## Step 7: Push Changes to Remote Repository
```bash
git push origin master
```

Output:

```
Writing objects: 100% (3/3), done.
To /opt/ecommerce.git
   418e307..2f7c705  master -> master
```

***

## Step 8: Verify Final Commit History
```bash
git log --oneline
```

Output:

```
2f7c705 Update info.txt
418e307 Add welcome.txt
b43cc6a initial commit
```

***

## Final Repository State
#### master branch contains:
* `initial commit`
* `Add welcome.txt`
* `Update info.txt`

#### feature branch contains:
* `Update welcome.txt`
* `Update info.txt`
* previous commits

***

## Key Concept Used
#### `git cherry-pick`
Cherry-pick allows you to apply a specific commit from one branch into another without merging the entire branch.

Syntax:

```bash
git cherry-pick <commit-hash>
```

***

## Conclusion
The requested commit `Update info.txt` was successfully:

* Identified from `feature`
* Cherry-picked into `master`
* Pushed to remote repository
* Verified in commit history

Task completed successfully.

## Common Mistake: Cherry-Picking the Wrong Commit (Educational Section)
During the process, it is possible to accidentally cherry-pick the wrong commit.

### What Went Wrong?
The required commit message was:

```bash
Update info.txt
```

But the following commit was mistakenly cherry-picked:

```bash
beb11d8 Update welcome.txt
```

The correct commit should have been:

```bash
2bb682f Update info.txt
```

***

## How to Identify the Mistake
After cherry-picking and pushing, checking the log revealed:

```bash
git log --oneline
```

Output:

```bash
e4c664c Update welcome.txt
418e307 Add welcome.txt
b43cc6a initial commit
```

This confirmed that the wrong commit had been merged into `master`.

***

## Fix It Properly
Since the incorrect commit was already pushed to the remote repository, we must:

1. Reset `master` back to the previous correct commit
2. Cherry-pick the correct commit
3. Force push (because history is rewritten)

***

### Step 1: Reset Master Back
Reset to the commit before the wrong cherry-pick:

```bash
git reset --hard 418e307
```

Output:

```bash
HEAD is now at 418e307 Add welcome.txt
```

***

### Step 2: Cherry-Pick the Correct Commit
```bash
git cherry-pick 2bb682f
```

Output:

```bash
[master 2f7c705] Update info.txt
 1 file changed, 1 insertion(+), 1 deletion(-)
```

***

### Step 3: Force Push the Correct History
Because the commit history changed, use force push:

```bash
git push origin master --force
```

Output:

```bash
+ e4c664c...2f7c705 master -> master (forced update)
```

***

### Step 4: Verify Final State
```bash
git log --oneline
```

Expected Output:

```bash
2f7c705 Update info.txt
418e307 Add welcome.txt
b43cc6a initial commit
```

***

## Final State Required
The `master` branch must contain:

* `initial commit`
* `Add welcome.txt`
* `Update info.txt`

And must NOT contain:

* `Update welcome.txt`

***

## Learning Outcome
This scenario demonstrates:

* The importance of verifying commit hashes before cherry-picking
* How to safely recover from pushing an incorrect commit
* Proper usage of `git reset --hard`
* When and why `git push --force` is required
