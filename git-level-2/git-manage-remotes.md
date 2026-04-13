# Git Manage Remotes

The xFusionCorp development team added updates to the project that is maintained under `/opt/cluster.git` repo and cloned under `/usr/src/kodekloudrepos/cluster`. Recently some changes were made on Git server that is hosted on `Storage server` in `Stratos DC`. The DevOps team added some new Git remotes, so we need to update remote on `/usr/src/kodekloudrepos/cluster` repository as per details mentioned below:

a. In `/usr/src/kodekloudrepos/cluster` repo add a new remote `dev_cluster` and point it to `/opt/xfusioncorp_cluster.git` repository.

b. There is a file `/tmp/index.html` on same server; copy this file to the repo and add/commit to master branch.

c. Finally push `master` branch to this new remote origin.

```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.247.241)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password: 
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/cluster
[natasha@ststor01 cluster]$ sudo git remote -v
origin  /opt/cluster.git (fetch)
origin  /opt/cluster.git (push)
[natasha@ststor01 cluster]$ git config --global --add safe.directory /usr/src/kodekloudrepos/cluster
[natasha@ststor01 cluster]$ sudo git remote add dev_cluster /opt/xfusioncorp_cluster.git
[natasha@ststor01 cluster]$ sudo git remote -vdev_cluster     /opt/xfusioncorp_cluster.git (fetch)
dev_cluster     /opt/xfusioncorp_cluster.git (push)
origin  /opt/cluster.git (fetch)
origin  /opt/cluster.git (push)
[natasha@ststor01 cluster]$ sudo cp /tmp/index.html .
[natasha@ststor01 cluster]$ sudo git checkout master
Already on 'master'
Your branch is up to date with 'origin/master'.
[natasha@ststor01 cluster]$ sudo git add index.html
[natasha@ststor01 cluster]$ sudo git commit -m "Added index.html file for xfusioncorp"
[master bbba600] Added index.html file for xfusioncorp
 1 file changed, 10 insertions(+)
 create mode 100644 index.html
[natasha@ststor01 cluster]$ sudo git push dev_cluster master
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 16 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 600 bytes | 600.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/xfusioncorp_cluster.git
 * [new branch]      master -> master
[natasha@ststor01 cluster]$ sudo git remote -v
dev_cluster     /opt/xfusioncorp_cluster.git (fetch)
dev_cluster     /opt/xfusioncorp_cluster.git (push)
origin  /opt/cluster.git (fetch)
origin  /opt/cluster.git (push)
[natasha@ststor01 cluster]$ sudo git branch -a
* master
  remotes/dev_cluster/master
  remotes/origin/master
[natasha@ststor01 cluster]$ ls
index.html  info.txt
[natasha@ststor01 cluster]$ 
```

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
