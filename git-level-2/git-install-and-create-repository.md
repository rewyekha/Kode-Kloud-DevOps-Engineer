# Git Install and Create Repository

The Nautilus development team shared with the DevOps team requirements for new application development, setting up a Git repository for that project. Create a Git repository on `Storage server` in Stratos DC as per details given below:

1. Install `git` package using `yum` on `Storage server`.
2. After that, create/init a git repository named `/opt/apps.git` (use the exact name as asked and make sure not to create a bare repository).



```bash
thor@jumphost ~$ ssh natasha@ststor01
The authenticity of host 'ststor01 (10.244.13.59)' can't be established.
ED25519 key fingerprint is SHA256:yEyN8qvzhNxfcKVE+H05zwQPmQMKCXj4JyGWuOP1HIg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ststor01' (ED25519) to the list of known hosts.
natasha@ststor01's password: 
[natasha@ststor01 ~]$ sudo yum install git
Last metadata expiration check: 0:04:00 ago on Mon Apr 13 05:28:17 2026.
Dependencies resolved.
===========================================================================================================================
 Package                          Architecture           Version                           Repository                 Size
===========================================================================================================================
Installing:
 git                              x86_64                 2.52.0-1.el9                      appstream                  39 k
Installing dependencies:
 git-core                         x86_64                 2.52.0-1.el9                      appstream                 5.0 M
 git-core-doc                     noarch                 2.52.0-1.el9                      appstream                 3.1 M
 less                             x86_64                 590-6.el9                         baseos                    162 k
 perl-Error                       noarch                 1:0.17029-7.el9                   appstream                  42 k
 perl-Git                         noarch                 2.52.0-1.el9                      appstream                  37 k
 perl-TermReadKey                 x86_64                 2.38-11.el9                       appstream                  37 k
 perl-lib                         x86_64                 0.65-483.el9                      appstream                  15 k

Transaction Summary
===========================================================================================================================
Install  8 Packages

Total download size: 8.5 M
Installed size: 43 M
Is this ok [y/N]: Y
Downloading Packages:
(1/8): git-2.52.0-1.el9.x86_64.rpm                                                         237 kB/s |  39 kB     00:00    
(2/8): less-590-6.el9.x86_64.rpm                                                           790 kB/s | 162 kB     00:00    
(3/8): perl-Error-0.17029-7.el9.noarch.rpm                                                 267 kB/s |  42 kB     00:00    
(4/8): perl-Git-2.52.0-1.el9.noarch.rpm                                                    431 kB/s |  37 kB     00:00    
(5/8): perl-TermReadKey-2.38-11.el9.x86_64.rpm                                             730 kB/s |  37 kB     00:00    
(6/8): perl-lib-0.65-483.el9.x86_64.rpm                                                    372 kB/s |  15 kB     00:00    
(7/8): git-core-2.52.0-1.el9.x86_64.rpm                                                    6.9 MB/s | 5.0 MB     00:00    
(8/8): git-core-doc-2.52.0-1.el9.noarch.rpm                                                5.2 MB/s | 3.1 MB     00:00    
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                      4.8 MB/s | 8.5 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                   1/1 
  Installing       : perl-lib-0.65-483.el9.x86_64                                                                      1/8 
  Installing       : perl-TermReadKey-2.38-11.el9.x86_64                                                               2/8 
  Installing       : perl-Error-1:0.17029-7.el9.noarch                                                                 3/8 
  Installing       : less-590-6.el9.x86_64                                                                             4/8 
  Installing       : git-core-2.52.0-1.el9.x86_64                                                                      5/8 
  Installing       : git-core-doc-2.52.0-1.el9.noarch                                                                  6/8 
  Installing       : perl-Git-2.52.0-1.el9.noarch                                                                      7/8 
  Installing       : git-2.52.0-1.el9.x86_64                                                                           8/8 
  Running scriptlet: git-2.52.0-1.el9.x86_64                                                                           8/8 
  Verifying        : less-590-6.el9.x86_64                                                                             1/8 
  Verifying        : git-2.52.0-1.el9.x86_64                                                                           2/8 
  Verifying        : git-core-2.52.0-1.el9.x86_64                                                                      3/8 
  Verifying        : git-core-doc-2.52.0-1.el9.noarch                                                                  4/8 
  Verifying        : perl-Error-1:0.17029-7.el9.noarch                                                                 5/8 
  Verifying        : perl-Git-2.52.0-1.el9.noarch                                                                      6/8 
  Verifying        : perl-TermReadKey-2.38-11.el9.x86_64                                                               7/8 
  Verifying        : perl-lib-0.65-483.el9.x86_64                                                                      8/8 

Installed:
  git-2.52.0-1.el9.x86_64                   git-core-2.52.0-1.el9.x86_64            git-core-doc-2.52.0-1.el9.noarch      
  less-590-6.el9.x86_64                     perl-Error-1:0.17029-7.el9.noarch       perl-Git-2.52.0-1.el9.noarch          
  perl-TermReadKey-2.38-11.el9.x86_64       perl-lib-0.65-483.el9.x86_64           

Complete!
[natasha@ststor01 ~]$ sudo git init /opt/apps.git
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
Initialized empty Git repository in /opt/apps.git/.git/
[natasha@ststor01 ~]$ ls -la /opt/apps.git
total 12
drwxr-xr-x 3 root root 4096 Apr 13 05:33 .
drwxr-xr-x 1 root root 4096 Apr 13 05:33 ..
drwxr-xr-x 6 root root 4096 Apr 13 05:33 .git
[natasha@ststor01 ~]$ cat /opt/apps.git/config | grep bare
cat: /opt/apps.git/config: No such file or directory
[natasha@ststor01 ~]$ 
```



<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
