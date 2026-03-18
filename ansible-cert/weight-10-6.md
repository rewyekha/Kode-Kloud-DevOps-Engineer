# Weight: 10

The Nautilus DevOps team intends to test multiple Ansible playbooks across various app servers in the `Stratos DC`. Before proceeding, certain prerequisites must be addressed. Specifically, the team requires the establishment of a password-less SSH connection between the Ansible controller and the managed nodes. An assigned ticket outlines the task; please carry out the following details:

a. The `Jump host` serves as our Ansible controller, and the Ansible playbooks will be executed through the `thor` user from the jump host.

b. An inventory file, `/home/thor/playbook/inventory-t3q2`, is available on the `jump host`. Utilize this inventory file to perform an Ansible ping from the jump host to `App Server 3` and ensure the successful execution of the ping command.



```bash
thor@jump-host ~/playbook$ cat /home/thor/playbook/inventory-t3q2
[applications]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner

thor@jump-host ~/playbook$ ssh-keygen -t rsa -b 2048 -f /home/thor/.ssh/id_rsa -N ""
Generating public/private rsa key pair.
Your identification has been saved in /home/thor/.ssh/id_rsa
Your public key has been saved in /home/thor/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:77h8/SlthdQTuUwrFfDnwp2k8kxijmQPkMQRwiWdFj4 thor@jump-host
The key's randomart image is:
+---[RSA 2048]----+
|    .o+*=    ...o|
|     .==.     .= |
|      .E      ++=|
|        o    o+B+|
|        S+ + ++o+|
|        o.* * ...|
|         ..+ + . |
|       . o. o o. |
|        +o.  +o  |
+----[SHA256]-----+

thor@jump-host ~/playbook$ ssh-copy-id banner@stapp03
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
banner@stapp03's password: 
Number of key(s) added: 1
Now try logging into the machine, with: "ssh 'banner@stapp03'"
and check to make sure that only the key(s) you wanted were added.

thor@jump-host ~/playbook$ ssh banner@stapp03
Last login: Wed Mar 18 16:00:09 2026 from 10.244.196.8
[banner@stapp03 ~]$ exit
logout
Connection to stapp03 closed.

thor@jump-host ~/playbook$ nano /home/thor/playbook/inventory-t3q2
# Updated inventory for passwordless SSH
[applications]
stapp01 ansible_host=stapp01 ansible_user=tony
stapp02 ansible_host=stapp02 ansible_user=steve
stapp03 ansible_host=stapp03 ansible_user=banner

thor@jump-host ~/playbook$ ansible -i /home/thor/playbook/inventory-t3q2 stapp03 -m ping
stapp03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
thor@jump-host ~/playbook$ 
```
