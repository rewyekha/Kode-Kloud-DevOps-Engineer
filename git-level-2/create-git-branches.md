# Create Git Branches

Nautilus developers are actively working on one of the project repositories, `/usr/src/kodekloudrepos/official`. Recently, they decided to implement some new features in the application, and they want to maintain those new changes in a separate branch. Below are the requirements that have been shared with the DevOps team:

1. On `Storage server` in Stratos DC create a new branch `xfusioncorp_official` from `master` branch in `/usr/src/kodekloudrepos/official` git repo.
2. Please do not try to make any changes in the code.



<pre class="language-bash"><code class="lang-bash"><strong>[natasha@ststor01 official]$ git config --global --add safe.directory /usr/src/kodekloudrepos/official
</strong><strong>[natasha@ststor01 official]$ git branch
</strong>* kodekloud_official
  master
<strong>[natasha@ststor01 official]$ git checkout -b xfusioncorp_official master
</strong>fatal: Unable to create '/usr/src/kodekloudrepos/official/.git/index.lock': Permission denied
<strong>[natasha@ststor01 official]$ sudo git checkout -b xfusioncorp_official master
</strong>Switched to a new branch 'xfusioncorp_official'
<strong>[natasha@ststor01 official]$ git branch
</strong>  kodekloud_official
  master
* xfusioncorp_official
<strong>[natasha@ststor01 official]$ git status
</strong>On branch xfusioncorp_official
nothing to commit, working tree clean
[natasha@ststor01 official]$ 
</code></pre>



<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
