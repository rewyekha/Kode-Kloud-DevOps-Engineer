# Day 34: Git Hook
Day 34: Git Hook

The Nautilus application development team was working on a git repository /opt/news.git which is cloned under /usr/src/kodekloudrepos directory present on Storage server in Stratos DC. The team want to setup a hook on this repository, please find below more details:

   Merge the feature branch into the master branch, but before pushing your changes complete below point.

   Create a post-update hook in this git repository so that whenever any changes are pushed to the master branch, it creates a release tag with name release-2023-06-15, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20. Make sure you test the hook at least once and create a release tag for today's release.

   Finally remember to push your changes.
   Note: Perform this task using the natasha user, and ensure the repository or existing directory permissions are not altered.

## Day 34: Git Hook – Auto Release Tag on Push
### Task Overview
The Nautilus development team is working on a Git repository:

* **Bare Repository:** `/opt/news.git`
* **Cloned Repository:** `/usr/src/kodekloudrepos/news`
* **Server:** `ststor01`
* **User:** `natasha`

#### Objectives
1. Merge the `feature` branch into `master`
2. Create a **post-update hook** in `/opt/news.git`
3.  When changes are pushed to `master`, automatically create a release tag:

    ```
    release-YYYY-MM-DD
    ```
4. Test the hook at least once
5. Push changes
6. Do NOT modify repository or directory permissions
7. Perform task using **natasha** user

***

## Step 1: Connect to Storage Server
From jump host:

```bash
ssh natasha@172.16.238.15
```

***

## Step 2: Navigate to Cloned Repository
```bash
cd /usr/src/kodekloudrepos
ls
cd news
git status
```

Ensure working branch is `feature`.

***

## Step 3: Merge Feature into Master
#### Switch to master
```bash
git checkout master
```

#### Pull latest changes
```bash
git pull origin master
```

#### Merge feature branch
```bash
git merge feature
```

Example output:

```
Updating 1190c50..104aa24
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

 Feature successfully merged into master.

***

## Step 4: Create Post-Update Hook
Move to bare repository hooks directory:

```bash
cd /opt/news.git/hooks
```

Create the hook:

```bash
vi post-update
```

***

### Add Following Script
```bash
#!/bin/bash

for ref in "$@"
do
    if [ "$ref" = "refs/heads/master" ]; then
        DATE=$(date +%F)
        TAG="release-$DATE"

        git tag -f "$TAG"
    fi
done
```

#### Explanation
* `"$@"` → receives updated refs from post-update hook
* Checks if push was made to `master`
* Creates tag in format:

    ```
    release-YYYY-MM-DD
    ```

***

### Make Hook Executable
```bash
chmod +x post-update
```

 Do NOT change ownership or permissions of repository.

***

## Step 5: Push Changes to Trigger Hook
Return to working repository:

```bash
cd /usr/src/kodekloudrepos/news
```

Push master:

```bash
git push origin master
```

If no new changes exist, create a test commit:

```bash
echo "trigger" >> test.txt
git add .
git commit -m "Trigger hook"
git push origin master
```

***

## Step 6: Verify Tag Creation
Check in bare repository:

```bash
cd /opt/news.git
git tag
```

Example output:

```
release-2026-02-27
```

 Tag matches current date
 Hook successfully triggered

***

## Common Errors & Solutions
| Issue               | Cause                     | Solution                        |
| ------------------- | ------------------------- | ------------------------------- |
| Tag not created     | Hook not executable       | `chmod +x post-update`          |
| Wrong hook logic    | Used post-receive syntax  | Use `"$@"` loop for post-update |
| No tag after push   | No new commit pushed      | Create test commit and push     |
| Tag pushed manually | Not required in bare repo | Only `git tag` is needed        |

***

## Final Verification Checklist
* [x] Merged `feature` into `master`
* [x] Created `post-update` hook in `/opt/news.git/hooks`
* [x] Hook executable
* [x] Tag format `release-YYYY-MM-DD`
* [x] Hook tested at least once
* [x] Changes pushed
* [x] Task performed using `natasha`
* [x] No permission changes made

***

## Final Result
Whenever changes are pushed to the `master` branch, a release tag is automatically created in this format:

```
release-YYYY-MM-DD
```

The hook is fully functional and meets all task requirements.
