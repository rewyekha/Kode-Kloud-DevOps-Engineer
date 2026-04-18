# Git task: Delete Git Branch

> **Reyas Khan** | [reyaskhan.me](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)

The Nautilus developers are engaged in active development on one of the project repositories located at `/usr/src/kodekloudrepos/ecommerce`. During testing, several test branches were created, and now they require cleanup. Here are the requirements provided to the DevOps team:

On the `Storage server` in Stratos DC, delete a branch named `xfusioncorp_ecommerce` from the `/usr/src/kodekloudrepos/ecommerce` Git repository.


***

## Delete a Git Branch in Repository

### Question

The Nautilus developers are engaged in active development on one of the project repositories located at `/usr/src/kodekloudrepos/ecommerce`. During testing, several test branches were created, and now they require cleanup.

**Task:**\
On the Storage server in Stratos DC, delete a branch named **`xfusioncorp_ecommerce`** from the `/usr/src/kodekloudrepos/ecommerce` Git repository.

***

## Solution

### Step 1: Connect to the Storage Server

```bash
thor@jumphost ~$ ssh natasha@ststor01
```

#### Terminal Output

```bash
The authenticity of host 'ststor01 (10.244.73.171)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
```

***

### Step 2: Navigate to the Git Repository

```bash
cd /usr/src/kodekloudrepos/ecommerce
```

***

### Step 3: Check Existing Branches

```bash
git branch
```

#### Terminal Output

```bash
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/ecommerce'
To add an exception for this directory, call:

git config --global --add safe.directory /usr/src/kodekloudrepos/ecommerce
```

Because of repository ownership restrictions, run the command with **sudo**.

```bash
sudo git branch
```

#### Output

```bash
master
* xfusioncorp_ecommerce
```

***

### Step 4: Switch to the `master` Branch

Since we cannot delete the branch we are currently on, switch to the **master** branch.

```bash
sudo git checkout master
```

#### Output

```bash
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

***

### Step 5: Verify Branches

```bash
sudo git branch
```

#### Output

```bash
* master
  xfusioncorp_ecommerce
```

***

### Step 6: Delete the Branch

```bash
sudo git branch -d xfusioncorp_ecommerce
```

#### Output

```bash
Deleted branch xfusioncorp_ecommerce (was 409f259).
```

***

### Step 7: Verify Deletion

```bash
sudo git branch
```

#### Output

```bash
* master
```

***

## Final Result

The branch **`xfusioncorp_ecommerce`** has been successfully removed from the repository located at:

```
/usr/src/kodekloudrepos/ecommerce
```

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

---

*[Reyas Khan](https://reyaskhan.me) | [GitHub](https://github.com/rewyekha)*
