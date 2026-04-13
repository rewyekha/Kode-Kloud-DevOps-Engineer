# Git Revert Some Changes

The Nautilus application development team was working on a git repository `/usr/src/kodekloudrepos/media` present on `Storage server` in `Stratos DC`. However, they reported an issue with the recent commits being pushed to this repo. They have asked the DevOps team to revert repo HEAD to last commit. Below are more details about the task:

1. In `/usr/src/kodekloudrepos/media` git repository, revert the latest commit `( HEAD )` to the previous commit (JFYI the previous commit hash should be with `initial commit` message ).
2. Use `revert media` message (please use all small letters for commit message) for the new revert commit.

```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.195.53)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password: 
[natasha@ststor01 ~]$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.195.53)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? ^C
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/media
[natasha@ststor01 media]$ git config --global --add safe.directory /usr/src/kodekloudrepos/media
[natasha@ststor01 media]$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        media.txt

nothing added to commit but untracked files present (use "git add" to track)
[natasha@ststor01 media]$ sudo git log --oneline
bef565e (HEAD -> master, origin/master) add data.txt file
a683aab initial commit
[natasha@ststor01 media]$ sudo git revert HEAD --no-edit
[master 0b8a21e] Revert "add data.txt file"
 Date: Mon Apr 13 06:38:12 2026 +0000
 1 file changed, 1 insertion(+)
 create mode 100644 info.txt
[natasha@ststor01 media]$ sudo git commit --amend -m "revert media"
[master 5fa1b8d] revert media
 Date: Mon Apr 13 06:38:12 2026 +0000
 1 file changed, 1 insertion(+)
 create mode 100644 info.txt
[natasha@ststor01 media]$ sudo git log --oneline -5
5fa1b8d (HEAD -> master) revert media
bef565e (origin/master) add data.txt file
a683aab initial commit
[natasha@ststor01 media]$ sudo git push origin master
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Writing objects: 100% (3/3), 274 bytes | 274.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/media.git
   bef565e..5fa1b8d  master -> master
[natasha@ststor01 media]$ sudo git log --oneline -3
5fa1b8d (HEAD -> master, origin/master) revert media
bef565e add data.txt file
a683aab initial commit
[natasha@ststor01 media]$ 
```
