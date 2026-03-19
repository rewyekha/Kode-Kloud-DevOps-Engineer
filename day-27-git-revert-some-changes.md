# Day 27: Git Revert Some Changes
The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/cluster` present on `Storage server` in `Stratos DC`. However, they reported an issue with the recent commits being pushed to this repo. They have asked the DevOps team to revert repo HEAD to last commit. Below are more details about the task:

1. In `/usr/src/kodekloudrepos/cluster` git repository, revert the latest commit `( HEAD )` to the previous commit (JFYI the previous commit hash should be with `initial commit` message ).
2. Use `revert cluster` message (please use all small letters for commit message) for the new revert commit.

***

## Day 27: Revert Latest Commit in Git Repository
### Objective
Revert the latest commit (`HEAD`) in the repository:

```
/usr/src/kodekloudrepos/cluster
```

Create a new revert commit with the message:

```
revert cluster
```

***

### Step 1: Connect to Storage Server
Login from the jump host:

```bash
ssh natasha@ststor01
```

Output:

```bash
The authenticity of host 'ststor01 (172.16.238.15)' can't be established.
ED25519 key fingerprint is SHA256:LVtbMxSgCdQsdRyzdtxBTgDdVgHJDzrVZgI91xjgKAw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

Switch to root:

```bash
sudo -i
```

Output:

```bash
[root@ststor01 ~]#
```

***

### Step 2: Navigate to Repository
```bash
cd /usr/src/kodekloudrepos/cluster
```

***

### Step 3: Check Git Log
```bash
git log --oneline
```

Output:

```bash
d1c53d9 (HEAD -> master, origin/master) add data.txt file
8b03eac initial commit
```

#### Observation
* `d1c53d9` → Latest commit (HEAD)
* `8b03eac` → Initial commit

We need to revert `d1c53d9`.

***

### Step 4: Revert Latest Commit
Run:

```bash
git revert HEAD -m 1
```

When prompted for commit message, replace it with:

```
revert cluster
```

After saving and exiting the editor, you will see:

```bash
[master 2bf87d1] revert cluster
 1 file changed, 1 insertion(+)
 create mode 100644 info.txt
```

***

### Step 5: Verify Changes
```bash
git log --oneline
```

Output:

```bash
2bf87d1 (HEAD -> master) revert cluster
d1c53d9 (origin/master) add data.txt file
8b03eac initial commit
```

***

## Final Result
Latest commit successfully reverted
New commit created with message: `revert cluster`
Repository history preserved

***

## Key Concept: `git revert` vs `git reset`
| Command      | Effect                                            |
| ------------ | ------------------------------------------------- |
| `git revert` | Creates a new commit that undoes previous changes |
| `git reset`  | Moves branch pointer backward (rewrites history)  |

For shared repositories, **`git revert` is the safe option**.

***

**Task Completed Successfully**
