# Weight: 17

## Configure Git to Ignore a File

### Question

Sarah created a new file named `notes-t1q9.txt` under the `/home/sarah/story-blog-t1q9` repository where she plans to write down ideas about the story for personal purposes.

She does **not want Git to track this file or share it with her teammates**.

Even though the file is currently untracked, if someone runs `git add .`, Git may start tracking it.

Configure Git so that the file is **ignored permanently**.

***

## Solution

We will configure a `.gitignore` file so that Git ignores `notes-t1q9.txt`.

***

## Step 1: SSH into the Storage Server

Login from the jump host to the storage server.

```bash
ssh natasha@ststor01.stratos.xfusioncorp.com
```

#### Terminal Output

```bash
thor@jumphost ~$ ssh natasha@ststor01.stratos.xfusioncorp.com
The authenticity of host 'ststor01.stratos.xfusioncorp.com (10.244.73.177)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
natasha@ststor01.stratos.xfusioncorp.com's password:
```

***

## Step 2: Switch to Sarah User

```bash
sudo su - sarah
```

#### Terminal Output

```bash
[natasha@ststor01 ~]$ sudo su - sarah
[sarah@ststor01 ~]$
```

***

## Step 3: Navigate to the Repository

```bash
cd /home/sarah/
ls
```

#### Terminal Output

```bash
[sarah@ststor01 ~]$ ls
story-blog-t1q1  story-blog-t1q10  story-blog-t1q5  story-blog-t1q6  story-blog-t1q8  story-blog-t1q9
```

Move into the required repository:

```bash
cd /home/sarah/story-blog-t1q9
ls
```

#### Terminal Output

```bash
[sarah@ststor01 story-blog-t1q9]$ ls
lion-and-mouse-t1q9.txt  notes-t1q9.txt
```

***

## Step 4: Add File to `.gitignore`

Add the file name to the `.gitignore` file.

```bash
echo "notes-t1q9.txt" >> .gitignore
```

Check the repository status.

```bash
git status
```

#### Terminal Output

```bash
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        .gitignore

nothing added to commit but untracked files present (use "git add" to track)
```

Notice that **`notes-t1q9.txt` does not appear anymore**, which means it is successfully ignored.

***

## Step 5: Commit `.gitignore`

Add and commit the `.gitignore` file.

```bash
git add .gitignore
git commit -m "Added gitignore to ignore personal notes file"
```

#### Terminal Output

```bash
[master 755c4be] Added gitignore to ignore personal notes file
 1 file changed, 1 insertion(+)
 create mode 100644 .gitignore
```

***

## Step 6: Verify Repository Status

```bash
git status
```

#### Terminal Output

```bash
On branch master
nothing to commit, working tree clean
```

***

## Final Result

* `notes-t1q9.txt` is **ignored permanently**
* `.gitignore` is **committed to the repository**
* Running `git add .` will **not track the notes file**

The task is successfully completed. 🎉
