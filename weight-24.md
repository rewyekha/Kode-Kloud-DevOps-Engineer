# Weight: 24

## Clone a Bare Git Repository

### Question

A developer recently created a **bare repository**. The team now wants to start adding content to it, so the repository must be cloned to a working directory.

**Task:**

Clone the repository:

```
/opt/story-blog-t1q11.git
```

into **Sarah's home directory** on the storage server.

**Credentials**

| Field    | Value     |
| -------- | --------- |
| Username | sarah     |
| Password | S3cure321 |

Finally, verify that the repository was cloned successfully.

***

## Solution

We will connect to the server and use Git to clone the bare repository.

***

## Step 1: SSH into the Storage Server

Login to the storage server.

```bash
ssh sarah@ststor01.stratos.xfusioncorp.com
```

#### Terminal Output

```bash
thor@jumphost ~$ ssh sarah@ststor01.stratos.xfusioncorp.com
sarah@ststor01.stratos.xfusioncorp.com's password:
Last login: Wed Mar 11
[sarah@ststor01 ~]$
```

***

## Step 2: Clone the Repository

Run the clone command from Sarah’s home directory.

```bash
git clone /opt/story-blog-t1q11.git
```

#### Terminal Output

```bash
[sarah@ststor01 ~]$ git clone /opt/story-blog-t1q11.git
Cloning into 'story-blog-t1q11'...
warning: You appear to have cloned an empty repository.
done.
```

***

## Step 3: Verify the Repository

Check if the repository directory exists.

```bash
ls
```

#### Terminal Output

```bash
[sarah@ststor01 ~]$ ls
story-blog-t1q1  story-blog-t1q10  story-blog-t1q11  story-blog-t1q5
story-blog-t1q6  story-blog-t1q8   story-blog-t1q9
```

Navigate inside the cloned repository.

```bash
cd story-blog-t1q11
ls -a
```

#### Terminal Output

```bash
[sarah@ststor01 story-blog-t1q11]$ ls -a
.  ..  .git
```

***

## Final Result

* The bare repository `/opt/story-blog-t1q11.git` has been successfully cloned.
* The repository is now available at:

```
/home/sarah/story-blog-t1q11
```

* Developers can now start adding content and pushing it to the remote repository.

✅ Task completed successfully.
