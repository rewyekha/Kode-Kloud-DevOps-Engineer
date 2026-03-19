# Day 33: Resolve Git Merge Conflicts
Sarah and Max were working on writting some stories which they have pushed to the repository. Max has recently added some new changes and is trying to push them to the repository but he is facing some issues. Below you can find more details:

SSH into `storage server` using user `max` and password `Max_pass123`. Under `/home/max` you will find the `story-blog` repository. Try to push the changes to the origin repo and fix the issues. The `story-index.txt` must have titles for all 4 stories. Additionally, there is a typo in `The Lion and the Mooose` line where `Mooose` should be `Mouse`.

Click on the `Gitea UI` button on the top bar. You should be able to access the `Gitea` page. You can login to `Gitea` server from UI using username `sarah` and password `Sarah_pass123` or username `max` and password `Max_pass123`.

`Note:` For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.

***

## Fix Push Rejection & Resolve Merge Conflict in Gitea Repository
### Problem Statement
Max attempted to push changes to the `story-blog` repository but encountered issues.

#### Requirements:
1. SSH into the storage server.
2. Navigate to `/home/max/story-blog`.
3. Ensure `story-index.txt` contains all **4 story titles**.
4. Fix the typo:
   * `Mooose`
   * `Mouse`
5. Push the changes to the remote repository.
6. Verify changes in Gitea UI.

***

## Infrastructure Details
| Server   | IP            | User    | Password |
| -------- | ------------- | ------- | -------- |
| ststor01 | 172.16.238.15 | natasha | Bl@kW    |

Repository Owner Credentials:

* **max / Max\_pass123**
* **sarah / Sarah\_pass123**

***

## Step 1: SSH into Storage Server
```bash
ssh natasha@172.16.238.15
```

Switch to max user:

```bash
sudo su - max
```

Navigate to repository:

```bash
cd /home/max/story-blog
```

***

## Step 2: Check Repository Status
```bash
git status
```

Output indicated:

```
Your branch is ahead of 'origin/master'
```

But push to `main` failed because branch name is `master`.

***

## Initial Push Error
```bash
git push origin main
```

Error:

```
error: src refspec main does not match any
```

#### Root Cause:
The repository uses **master branch**, not `main`.

***

## Correct Push Command
```bash
git push origin master
```

***

## Second Error: Rejected Push
```
! [rejected] master -> master (fetch first)
```

#### Root Cause:
Remote repository had new commits not present locally.

***

## Step 3: Rebase with Remote
```bash
git pull origin master --rebase
```

This resulted in a merge conflict in:

```
story-index.txt
```

***

## Merge Conflict Found
Conflict content:

```
<<<<<<< HEAD
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
=======
1. The Lion and the Mooose
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
>>>>>>> Added the fox and grapes story
```

***

## Step 4: Resolve Conflict
Edited `story-index.txt` to:

```
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

Removed:

* `<<<<<<<`
* `=======`
* `>>>>>>>`
* Incorrect spelling `Mooose`

***

## Step 5: Continue Rebase
```bash
git add story-index.txt
git rebase --continue
```

Rebase completed successfully.

***

## Step 6: Final Push
```bash
git push origin master
```

Successful output:

```
master -> master
```

***

## Verification in Gitea UI
Logged into Gitea as `max`.

Confirmed:

* All 4 story titles present
* Typo fixed
* No conflict markers
* Latest commit visible

***

***

## Final Outcome
 Branch mismatch issue resolved
 Rebase completed
 Merge conflict fixed
 Typo corrected
 Changes successfully pushed
 Verified in Gitea

***
