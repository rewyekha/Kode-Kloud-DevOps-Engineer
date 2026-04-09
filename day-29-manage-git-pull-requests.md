# Day 29: Manage Git Pull Requests

`Max` want to push some new changes to one of the repositories but we don't want people to push directly to `master` branch, since that would be the final version of the code. It should always only have content that has been reviewed and approved. We cannot just allow everyone to directly push to the master branch. So, let's do it the right way as discussed below:

SSH into `storage server` using user `max`, password `Max_pass123` . There you can find an already cloned repo under `Max` user's home.

Max has written his story about The 🦊 Fox and Grapes 🍇

Max has already pushed his story to remote git repository hosted on `Gitea` branch `story/fox-and-grapes`

Check the contents of the cloned repository. Confirm that you can see Sarah's story and history of commits by running `git log` and validate author info, commit message etc.

Max has pushed his story, but his story is still not in the `master` branch. Let's create a Pull Request(PR) to merge Max's `story/fox-and-grapes` branch into the `master` branch

Click on the `Gitea UI` button on the top bar. You should be able to access the `Gitea` page.

UI login info:

\- Username: `max`

\- Password: `Max_pass123`

PR title : `Added fox-and-grapes story`

PR pull from branch: `story/fox-and-grapes` (source)

PR merge into branch: `master` (destination)

Before we can add our story to the `master` branch, it has to be reviewed. So, let's ask `tom` to review our PR by assigning him as a reviewer\
Add tom as reviewer through the Git Portal UI

* Go to the newly created PR
* Click on Reviewers on the right
* Add tom as a reviewer to the PR

Now let's review and approve the PR as user `Tom`\
Login to the portal with the user `tom`

Logout of `Git Portal UI` if logged in as `max`

UI login info:

\- Username: `tom`

\- Password: `Tom_pass123`

PR title : `Added fox-and-grapes story`

Review and merge it.

Great stuff!! The story has been merged! 👏

`Note:` For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.



## ✅ Step 1: SSH into Storage Server as Max

From the jump host (if required), SSH into the storage server:

```bash
ssh max@ststor01.stratos.xfusioncorp.com
```

Password:

```
Max_pass123
```

***

## ✅ Step 2: Verify the Cloned Repository

Navigate to Max’s home directory and locate the cloned repository:

```bash
cd ~
ls
```

Enter the repo directory:

```bash
cd <repository-name>
```

Check repository contents:

```bash
ls -la
```

You should see Sarah’s story files.

***

## ✅ Step 3: Validate Commit History

Run:

```bash
git log --oneline --decorate --graph
```

For detailed commit info:

```bash
git log
```

Confirm:

* Author name (e.g., Sarah or Max)
* Commit message
* Branch information
* Commit timestamps

You should see:

* Sarah’s earlier commits on `master`
* Max’s commit on branch `story/fox-and-grapes`

Also verify current branch:

```bash
git branch
```

***

## ✅ Step 4: Confirm Max’s Story Branch Exists Remotely

```bash
git branch -r
```

You should see:

```
origin/story/fox-and-grapes
origin/master
```

This confirms Max already pushed his branch.

***

## ✅ Step 5: Create Pull Request in Gitea UI

#### Open Gitea Web UI

Click **Gitea** from the top bar.

Login as:

* **Username:** max
* **Password:** Max\_pass123

***

#### Create Pull Request

1. Go to the repository.
2. Click **Pull Requests**
3. Click **New Pull Request**

Set:

* **Title:** `Added fox-and-grapes story`
* **Source branch:** `story/fox-and-grapes`
* **Destination branch:** `master`

Click **Create Pull Request**

***

## ✅ Step 6: Assign Reviewer (Tom)

Inside the newly created PR:

1. On the right side, click **Reviewers**
2. Add **tom** as reviewer
3. Save changes

***

## ✅ Step 7: Review and Merge as Tom

#### Logout Max

Click profile → Logout.

#### Login as Tom

* **Username:** tom
* **Password:** Tom\_pass123

***

#### Review PR

1. Open the PR titled:\
   **Added fox-and-grapes story**
2. Review file changes
3. Click **Review Changes**
4. Choose **Approve**
5. Submit review

***

## ✅ Step 8: Merge the Pull Request

After approval:

1. Click **Merge Pull Request**
2. Confirm merge

You should now see:

```
Merged
```

***

## ✅ Step 9: Verify Merge (Optional CLI Validation)

SSH back into storage server and pull latest changes:

```bash
git checkout master
git pull origin master
```

Verify story is now in master:

```bash
cat <story-file>
```

Check log:

```bash
git log --oneline
```

You should now see Max’s commit in master branch history.

***

## 🎉 Expected Final Result

* Direct push to master prevented
* PR created properly
* Reviewer assigned
* PR approved by Tom
* Story successfully merged into master
* Proper Git workflow followed

***

<figure><img src=".gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>
