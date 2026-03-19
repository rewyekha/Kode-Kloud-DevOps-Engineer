# Day 32: Git Rebase
The Nautilus application development team has been working on a project repository `/opt/apps.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:

One of the developers is working on `feature` branch and their work is still in progress, however there are some changes which have been pushed into the `master` branch, the developer now wants to `rebase` the `feature` branch with the `master` branch without loosing any data from the `feature` branch, also they don't want to add any `merge commit` by simply merging the `master` branch into the `feature` branch. Accomplish this task as per requirements mentioned.

Also remember to push your changes once done.

## GitBook Documentation
## Day 32 – Git Rebase Without Merge Commit
***

### Scenario
The Nautilus application development team is working on a repository:

```
/opt/apps.git
```

The repository is cloned on the storage server at:

```
/usr/src/kodekloudrepos
```

#### Requirement
A developer is working on the `feature` branch. Meanwhile, new changes were pushed to `master`.

The developer wants to:

* Rebase `feature` branch with `master`
* Avoid losing any feature branch work
* Avoid creating a merge commit
* Push the updated branch

***

## Infrastructure Details
Server: `ststor01`
User: `natasha`
Repository location: `/usr/src/kodekloudrepos`

***

## Step-by-Step Solution
***

### Step 1: SSH into Storage Server
```bash
thor@jumphost ~$ ssh natasha@ststor01.stratos.xfusioncorp.com
natasha@ststor01.stratos.xfusioncorp.com's password:
```

Switch to root:

```bash
[natasha@ststor01 ~]$ sudo -i
[root@ststor01 ~]#
```

***

### Mistake #1: Running Git in Wrong Directory
Initially, Git commands were run here:

```bash
[root@ststor01 ~]# cd /usr/src/kodekloudrepos
[root@ststor01 kodekloudrepos]# git branch
fatal: not a git repository (or any of the parent directories): .git
```

#### Why This Happened?
Because `/usr/src/kodekloudrepos` was **not the actual git repository root**.

Checking contents:

```bash
[root@ststor01 kodekloudrepos]# ls -la
total 12
drwxr-xr-x 3 root root 4096 Feb 25 03:58 .
drwxr-xr-x 1 root root 4096 Feb 25 03:58 ..
drwxr-xr-x 3 root root 4096 Feb 25 03:58 apps
```

The actual repository was inside the `apps` directory.

***

### Step 2: Move to Correct Repository Directory
```bash
[root@ststor01 kodekloudrepos]# cd apps
[root@ststor01 apps]#
```

Verify branches:

```bash
[root@ststor01 apps]# git branch
* feature
  master
```

Now Git works correctly.

***

### Step 3: Fetch Latest Changes
```bash
[root@ststor01 apps]# git fetch origin
```

Verify remote branches:

```bash
[root@ststor01 apps]# git branch -a
* feature
  master
  remotes/origin/feature
  remotes/origin/master
```

***

### Step 4: Ensure On Feature Branch
```bash
[root@ststor01 apps]# git checkout feature
Already on 'feature'
```

***

### Step 5: Rebase Feature Onto Master
```bash
[root@ststor01 apps]# git rebase origin/master
Successfully rebased and updated refs/heads/feature.
```

#### What This Did
* Took feature commits
* Replayed them on top of latest `origin/master`
* Avoided creating a merge commit
* Maintained clean linear history

***

### Step 6: Verify Working Tree
```bash
[root@ststor01 apps]# git status
On branch feature
nothing to commit, working tree clean
```

***

### Step 7: Push Rebased Branch
Since rebase rewrites commit history, a force push is required.

```bash
[root@ststor01 apps]# git push origin feature --force
Enumerating objects: 4, done.
Writing objects: 100% (3/3), done.
To /opt/apps.git
 + 3db22ab...f46aeda feature -> feature (forced update)
```

***

### Step 8: Final Verification
```bash
[root@ststor01 apps]# git log --oneline --graph --decorate --all
* f46aeda (HEAD -> feature, origin/feature) Add new feature
* 17cc18d (origin/master, master) Update info.txt
* 516c915 initial commit
```

***

## Final Result
 Feature branch rebased successfully
 No merge commit created
 No data loss
 Remote branch updated
 Clean linear history

***

## Key Concepts Explained
### Rebase vs Merge
| Merge                      | Rebase                  |
| -------------------------- | ----------------------- |
| Creates merge commit       | No merge commit         |
| Non-linear history         | Linear history          |
| Easier for shared branches | Cleaner project history |

***

## Common Mistakes in This Task
#### 1 Running Git Outside Repository
Error:

```
fatal: not a git repository
```

Fix:
Navigate into the directory containing `.git`.

***

#### 2 Forgetting to Fetch Before Rebase
If you don’t run:

```bash
git fetch origin
```

You may rebase against outdated local master.

***

#### 3 Forgetting to Force Push After Rebase
Rebase rewrites commit history.

Without:

```bash
git push --force
```

Push will fail.

***

#### 4 Rebasing the Wrong Branch
Always confirm:

```bash
git branch
```

The `*` must be on `feature`.

***

## Best Practices
* Always run `git status` before rebasing
* Always fetch latest changes
* Avoid rebasing shared public branches
* Use `--force` carefully

***

## Conclusion
The `feature` branch was successfully rebased onto `master` without:

* Losing work
* Creating a merge commit

This ensures a clean, professional Git history suitable for production environments.

***

 **End of Day 32 – Git Rebase Documentation**
