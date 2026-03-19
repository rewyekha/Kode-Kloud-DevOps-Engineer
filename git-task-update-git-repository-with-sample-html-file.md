# Git task: Update Git Repository with Sample HTML File
## Copying `index.html` to Git Repository and Pushing to Master
### Question
The Nautilus development team has created a new repository `/opt/media.git` for a project. You are provided with a sample `index.html` file located on the jump host under `/tmp`. The repository has been cloned to `/usr/src/kodekloudrepos` on the storage server (`ststor01`).

**Task:**

1. Copy the `index.html` file from the jump host to the storage server inside the cloned repository at `/usr/src/kodekloudrepos/media`.
2. Add and commit the file to the repository.
3. Push the changes to the `master` branch.

**Infrastructure Details:**

* Jump Host: `jump_host.stratos.xfusioncorp.com` (`thor` / `mjolnir123`)
* Storage Server: `ststor01.stratos.xfusioncorp.com` (`natasha` / `Bl@kW`)
* Repository Path: `/usr/src/kodekloudrepos/media`

***

### File Flow Diagram
```mermaid
flowchart LR
    A[Jump Host<br>/tmp/index.html] -->|scp| B[Storage Server<br>/tmp/index.html]
    B -->|sudo cp| C[Cloned Git Repo<br>/usr/src/kodekloudrepos/media/index.html]
    C -->|git add + commit + push| D[Remote Repository<br>/opt/media.git<br>master branch]
```

**Explanation:**

1. The file `index.html` is first located on the **Jump Host** under `/tmp`.
2. It is copied using `scp` to the **Storage Server** `/tmp`.
3. On the Storage Server, it is moved into the cloned repository folder `/usr/src/kodekloudrepos/media`.
4. The file is added, committed, and pushed to the **master branch** of the remote repository `/opt/media.git`.

***

### Step-by-Step Solution with Terminal Output
#### 1. SSH to the Jump Host
```bash
thor@jumphost ~$ ssh thor@jump_host.stratos.xfusioncorp.com
The authenticity of host 'jump_host.stratos.xfusioncorp.com (172.16.238.3)' can't be established.
ED25519 key fingerprint is SHA256:CtkNUSeULzponMTDYeK2sO3tvk4fTOjVLMammU5ql0M.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'jump_host.stratos.xfusioncorp.com' (ED25519) to the list of known hosts.
thor@jump_host.stratos.xfusioncorp.com's password:
Last login: Thu Mar  5 03:43:05 2026
```

***

#### 2. Copy the File to Storage Server
```bash
thor@jump_host ~$ scp /tmp/index.html natasha@ststor01:/tmp
The authenticity of host 'ststor01 (172.16.238.15)' can't be established.
ED25519 key fingerprint is SHA256:/g4PSJlUhByUaC8kI5ZjtGimI69nE5U2tzzudkTAXlM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password:
index.html                                                                                                  100%   27   126.7KB/s   00:00
```

***

#### 3. SSH to the Storage Server
```bash
thor@jump_host ~$ ssh natasha@ststor01
natasha@ststor01's password:
[natasha@ststor01 ~]$
```

***

#### 4. Move File into Repository
```bash
[natasha@ststor01 ~]$ cp /tmp/index.html /usr/src/kodekloudrepos/media/
cp: cannot create regular file '/usr/src/kodekloudrepos/media/index.html': Permission denied
[natasha@ststor01 ~]$ sudo cp /tmp/index.html /usr/src/kodekloudrepos/media/

[sudo] password for natasha:
```

***

#### 5. Navigate to the Repository
```bash
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/media
```

***

#### 6. Fix Git “Dubious Ownership” Warning
```bash
[natasha@ststor01 media]$ git status
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/media'
To add an exception for this directory, call:
        git config --global --add safe.directory /usr/src/kodekloudrepos/media

[natasha@ststor01 media]$ git config --global --add safe.directory /usr/src/kodekloudrepos/media
```

***

#### 7. Add, Commit, and Push the File
```bash
[natasha@ststor01 media]$ sudo git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  index.html

[natasha@ststor01 media]$ sudo git add index.html
[natasha@ststor01 media]$ sudo git commit -m "Added sample index.html"
[master 7fadf90] Added sample index.html
 1 file changed, 1 insertion(+)
 create mode 100644 index.html

[natasha@ststor01 media]$ sudo git push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 337 bytes | 337.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/media.git
   8b9ee08..7fadf90  master -> master

[natasha@ststor01 media]$ sudo git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

 The `index.html` file is successfully added, committed, and pushed to the master branch
