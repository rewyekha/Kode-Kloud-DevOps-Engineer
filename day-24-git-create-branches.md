# Day 24: Git Create Branches

Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/news`. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:

1. On `Storage server` in Stratos DC create a new branch `xfusioncorp_news` from `master` branch in `/usr/src/kodekloudrepos/news` git repo.
2. Please do not try to make any changes in the code.

## Create New Branch `xfusioncorp_news` from `master`

### 📌 Task

Create a new branch **`xfusioncorp_news`** from the `master` branch in the repository:

```
/usr/src/kodekloudrepos/news
```

⚠️ Do **not** make any code changes.

***

### 🖥 Server Details

* **Server:** `ststor01`
* **User:** `natasha`
* **Repository Path:** `/usr/src/kodekloudrepos/news`

***

### 🚀 Step-by-Step Solution

#### 1️⃣ SSH into Storage Server

From jump host:

```bash
ssh natasha@ststor01
```

Enter password when prompted.

***

#### 2️⃣ Navigate to Repository

```bash
cd /usr/src/kodekloudrepos/news
```

***

#### 3️⃣ Resolve Dubious Ownership Issue

When running `git status`, you may see:

```
fatal: detected dubious ownership in repository
```

Instead of marking directory as safe, switch to root user (recommended approach for this lab):

```bash
sudo -i
```

Enter password for `natasha`.

***

#### 4️⃣ Navigate to Repository as Root

```bash
cd /usr/src/kodekloudrepos/news
```

Verify repository:

```bash
git status
git branch
```

***

#### 5️⃣ Switch to master Branch

```bash
git checkout master
```

Confirm:

```bash
git branch
```

Output should show:

```
* master
```

***

#### 6️⃣ Create New Branch

```bash
git checkout -b xfusioncorp_news
```

***

#### 7️⃣ Verify Branch Creation

```bash
git branch
```

Expected Output:

```
  kodekloud_news
  master
* xfusioncorp_news
```

***

### ✅ Final Verification

```bash
git status
```

Output:

```
On branch xfusioncorp_news
nothing to commit, working tree clean
```

✔ Branch successfully created\
✔ No code changes made\
✔ Working tree clean

***

### 📚 Key Notes

* If repository ownership issues appear, switching to root resolves it.
* Always ensure you're on `master` before creating a new branch.
* Use `git branch` to confirm active branch.
* Do not modify any files as per requirement.

***

**Task Status: Completed Successfully** ✅

