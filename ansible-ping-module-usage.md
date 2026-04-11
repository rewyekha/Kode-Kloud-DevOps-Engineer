# Ansible Ping Module Usage

The Nautilus DevOps team is planning to test several Ansible playbooks on different app servers in `Stratos DC`. Before that, some pre-requisites must be met. Essentially, the team needs to set up a password-less SSH connection between Ansible controller and Ansible managed nodes. One of the tickets is assigned to you; please complete the task as per details mentioned below:

a. `Jump host` is our Ansible controller, and we are going to run Ansible playbooks through `thor` user from `jump host`.

b. There is an inventory file `/home/thor/ansible/inventory` on `jump host`. Using that inventory file test `Ansible ping` from `jump host` to `App Server 1`, make sure ping works.



***

## Ansible Password-less SSH Setup and Connectivity Test

### Objective

Configure password-less SSH authentication between the Ansible controller (Jump Host) and the managed node (Application Server 1), and verify connectivity using the Ansible ping module.

***

### Environment Details

| Server Name          | Hostname  | User | Role               |
| -------------------- | --------- | ---- | ------------------ |
| Jump Host            | jump-host | thor | Ansible Controller |
| Application Server 1 | stapp01   | tony | Managed Node       |

***

### Inventory File Location

```
/home/thor/ansible/inventory
```

***

### Step 1: Navigate to Ansible Directory

```
thor@jump-host ~$ ls
ansible

thor@jump-host ~$ cd ansible

thor@jump-host ~/ansible$ ls
inventory
```

***

### Step 2: Verify Inventory File

```
thor@jump-host ~/ansible$ cat inventory
stapp01 ansible_ssh_pass=Ir0nM@n
stapp02 ansible_ssh_pass=Am3ric@
stapp03 ansible_ssh_pass=BigGr33n
```

***

### Step 3: Generate SSH Key on Jump Host

```
thor@jump-host ~/ansible$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/thor/.ssh/id_rsa): 
Enter passphrase for "/home/thor/.ssh/id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/thor/.ssh/id_rsa
Your public key has been saved in /home/thor/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ga0+znjzAqtuf8YxArvSFNAxu9tE1FDJ9/C88g3RZKg thor@jump-host
The key's randomart image is:
+---[RSA 3072]----+
| .o.o=..   .     |
|. .+  +oo . o    |
| .. . ..o* +     |
|  oo   .E.= .    |
|  .+. . S  o     |
|  o+o.o . o      |
| o...=oo o o     |
|. + .+B.  . .    |
| +ooo+o+.        |
+----[SHA256]-----+
```

***

### Step 4: Copy SSH Key to Application Server 1

```
thor@jump-host ~/ansible$ ssh-copy-id tony@stapp01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_rsa.pub"

The authenticity of host 'stapp01 (10.244.49.33)' can't be established.
ED25519 key fingerprint is SHA256:JZHSyjPyHefs3xPXAcYzGrs1MSYUKXe7EIDxqJsq2uw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

tony@stapp01's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'tony@stapp01'"
and check to make sure that only the key(s) you wanted were added.
```

***

### Step 5: Verify Password-less SSH Login

```
thor@jump-host ~/ansible$ ssh tony@stapp01
[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.
```

Successful login without password prompt confirms key-based authentication.

***

### Step 6: Update Inventory File

```
thor@jump-host ~/ansible$ vi ~/ansible/inventory
```

Updated content:

```
stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n
stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@
stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n
```

***

### Step 7: Test Ansible Connectivity

```
thor@jump-host ~/ansible$ ansible -i ~/ansible/inventory stapp01 -m ping
```

Output:

```
stapp01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

***

### Optional Verification Using Password

```
thor@jump-host ~/ansible$ ansible -i ~/ansible/inventory stapp01 -m ping --ask-pass
SSH password: 
```

Output:

```
stapp01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

***

### Final Outcome

* SSH key-based authentication successfully configured between Jump Host and Application Server 1
* Inventory file properly structured
* Ansible ping module successfully executed
* Connectivity between controller and managed node verified

***

### Conclusion

The Ansible controller is now configured to communicate securely with the managed node using password-less SSH. This setup enables seamless execution of Ansible playbooks across the infrastructure.

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

