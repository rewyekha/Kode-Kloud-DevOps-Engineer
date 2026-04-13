# Git Merge Branches

The Nautilus application development team has been working on a project repository `/opt/cluster.git`. This repo is cloned at `/usr/src/kodekloudrepos` on `storage server` in `Stratos DC`. They recently shared the following requirements with DevOps team:

Create a new branch `devops` in `/usr/src/kodekloudrepos/cluster` repo from `master` and copy the `/tmp/index.html` file (present on `storage server` itself) into the repo. Further, `add/commit` this file in the new branch and merge back that branch into `master` branch. Finally, push the changes to the origin for both of the branches.



<pre class="language-bash"><code class="lang-bash">thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.240.189)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password: 
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/cluster
[natasha@ststor01 cluster]$ git branch
fatal: detected dubious ownership in repository at '/usr/src/kodekloudrepos/cluster'
To add an exception for this directory, call:

        git config --global --add safe.directory /usr/src/kodekloudrepos/cluster
<strong>[natasha@ststor01 cluster]$ git config --global --add safe.directory /usr/src/kodekloudrepos/cluster
</strong><strong>[natasha@ststor01 cluster]$ git branch
</strong>* master
[natasha@ststor01 cluster]$ git checkout master
fatal: Unable to create '/usr/src/kodekloudrepos/cluster/.git/index.lock': Permission denied
<strong>[natasha@ststor01 cluster]$ sudo git checkout master
</strong>Already on 'master'
Your branch is up to date with 'origin/master'.
<strong>[natasha@ststor01 cluster]$ sudo git pull origin master
</strong>From /opt/cluster
 * branch            master     -> FETCH_HEAD
Already up to date.
[natasha@ststor01 cluster]$ git checkout -b devops
fatal: cannot lock ref 'refs/heads/devops': Unable to create '/usr/src/kodekloudrepos/cluster/.git/refs/heads/devops.lock': Permission denied
<strong>[natasha@ststor01 cluster]$ sudo git checkout -b devops
</strong>Switched to a new branch 'devops'
[natasha@ststor01 cluster]$ cp /tmp/index.html .
cp: cannot create regular file './index.html': Permission denied
<strong>[natasha@ststor01 cluster]$ sudo cp /tmp/index.html .
</strong><strong>[natasha@ststor01 cluster]$ sudo git add index.html
</strong><strong>[natasha@ststor01 cluster]$ sudo git commit -m "Added index.html file"
</strong>[devops 2c3076a] Added index.html file
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
<strong>[natasha@ststor01 cluster]$ sudo git push origin devops
</strong>Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 335 bytes | 335.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/cluster.git
 * [new branch]      devops -> devops
<strong>[natasha@ststor01 cluster]$ git checkout master
</strong>fatal: Unable to create '/usr/src/kodekloudrepos/cluster/.git/index.lock': Permission denied
<strong>[natasha@ststor01 cluster]$ sudo git checkout master
</strong>Switched to branch 'master'
Your branch is up to date with 'origin/master'.
<strong>[natasha@ststor01 cluster]$ git merge devops
</strong>fatal: update_ref failed for ref 'ORIG_HEAD': cannot lock ref 'ORIG_HEAD': Unable to create '/usr/src/kodekloudrepos/cluster/.git/ORIG_HEAD.lock': Permission denied
<strong>[natasha@ststor01 cluster]$ sudo git merge devops
</strong>Updating c22e432..2c3076a
Fast-forward
 index.html | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
<strong>[natasha@ststor01 cluster]$ sudo git push origin master
</strong>Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/cluster.git
   c22e432..2c3076a  master -> master
[natasha@ststor01 cluster]$ Bashgit branch -a
-bash: Bashgit: command not found
<strong>[natasha@ststor01 cluster]$ sudo git branch -a
</strong>  devops
* master
  remotes/origin/devops
  remotes/origin/master
<strong>[natasha@ststor01 cluster]$ sudo git log --oneline --graph
</strong>* 2c3076a (HEAD -> master, origin/master, origin/devops, devops) Added index.html file
* c22e432 initial commit
<strong>[natasha@ststor01 cluster]$ ls
</strong>index.html  info.txt  welcome.txt
[natasha@ststor01 cluster]$ 
</code></pre>



<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
