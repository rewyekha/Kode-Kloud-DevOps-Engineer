# Summary

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

***

## 📘 Ansible Hands-on Lab (KodeKloud)&#x20;

***

## 🧩 1/10 — Enable Sudo Password Prompt

### 📘 Topic

Ansible Configuration → Privilege Escalation

***

### 🎯 Goal

Make Ansible prompt for sudo password during execution.

***

### 🔧 Steps

#### 1. Open config

```bash
vi /home/thor/ansible-t5q4/ansible-t5q4.cfg
```

#### 2. Add/Update:

```ini
[defaults]
ask_become_pass = True

[privilege_escalation]
become = True
become_method = sudo
become_user = root
```

#### 3. Export config

```bash
export ANSIBLE_CONFIG=/home/thor/ansible-t5q4/ansible-t5q4.cfg
```

#### 4. Run playbook

```bash
ansible-playbook playbook.yml
```

***

### ✅ Output

```bash
BECOME password:
```

***

### 🧠 Key Concept

* `ask_become_pass=True` → enables sudo password prompt
* Required when sudo is not passwordless

***

## 🧩 2/10 — Configure Logging

### 📘 Topic

Ansible Configuration → Logging

***

### 🎯 Goal

Store logs in custom file

***

### 🔧 Steps

#### 1. Edit config

```bash
vi /home/thor/ansible-t5q3/ansible-t5q3.cfg
```

#### 2. Add:

```ini
[defaults]
log_path = /home/thor/ansible-t5q3/ansible-t5q3.log
```

#### 3. Export config

```bash
export ANSIBLE_CONFIG=/home/thor/ansible-t5q3/ansible-t5q3.cfg
```

***

### ✅ Output (after any playbook run)

```bash
ls /home/thor/ansible-t5q3/
ansible-t5q3.log
```

***

### 🧠 Key Concept

* Avoid `/var/log` → permission issue
* Use user-owned directory

***

## 🧩 3/10 — Copy File (Localhost)

### 📘 Topic

copy module

***

### 🎯 Goal

Copy file on same machine

***

### 🔧 Steps

#### 1. Create playbook

```bash
vi /home/thor/ansible/playbook-t2q3.yml
```

#### 2. Add:

```yaml
- hosts: localhost
  become: yes
  tasks:
    - name: Copy file locally
      copy:
        src: /usr/src/security-t2q3/linux-t2q3.txt
        dest: /opt/security-t2q3/linux-t2q3.txt
```

#### 3. Run

```bash
ansible-playbook -i localhost playbook-t2q3.yml
```

***

### ✅ Output

```bash
PLAY [localhost]

TASK [Copy file locally]
changed: [localhost]

PLAY RECAP
localhost : ok=2 changed=1 failed=0
```

***

### 🧠 Concept

* `copy` works locally when `hosts: localhost`
* `become` needed for `/opt`

***

## 🧩 4/10 — Copy File to Multiple Servers

### 📘 Topic

Inventory + copy module

***

### 🔧 Steps

#### 1. Check inventory

```bash
cat /home/thor/ansible/inventory-t2q1
```

Example:

```ini
[applications]
stapp01
stapp02
stapp03
```

***

#### 2. Create playbook

```bash
vi /home/thor/ansible/playbook-t2q1.yml
```

```yaml
- hosts: applications
  become: yes
  tasks:
    - name: Create dir
      file:
        path: /opt/security-t2q1
        state: directory

    - name: Copy file
      copy:
        src: /usr/src/security-t2q1/index-t2q1.html
        dest: /opt/security-t2q1/index-t2q1.html
```

***

#### 3. Run

```bash
ansible-playbook -i inventory-t2q1 playbook-t2q1.yml
```

***

### ✅ Output

```bash
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]
```

***

### 🧠 Concept

* Group name must match inventory
* Copy runs from control node → targets

***

## 🧩 5/10 — Create File

### 📘 Topic

file module

***

### 🔧 Steps

```bash
vi /home/thor/playbook/playbook-t4q1.yml
```

```yaml
- hosts: app_servers
  become: yes
  tasks:
    - file:
        path: /tmp/webdata-t4q1.txt
        state: touch
```

Run:

```bash
ansible-playbook -i inventory-t4q1 playbook-t4q1.yml
```

***

### ✅ Output

```bash
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]
```

***

### 🧠 Concept

* `touch` → create empty file

***

## 🧩 6/10 — Delete File

### 📘 Topic

file module

***

### 🔧 Steps

```bash
vi /home/thor/playbook/playbook-t4q4.yml
```

```yaml
- hosts: app_servers
  become: yes
  tasks:
    - file:
        path: /opt/fruits-t4q4.txt
        state: absent
```

Run:

```bash
ansible-playbook -i inventory-t4q4 playbook-t4q4.yml
```

***

### ✅ Output

```bash
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]
```

***

### 🧠 Concept

* `absent` → delete file

***

## 🧩 7/10 — Add Host to Inventory

### 📘 Topic

Inventory basics

***

### 🔧 Steps

```bash
vi /home/thor/playbook/inventory-t3q5
```

Add:

```ini
server4.company.com
```

***

### ✅ Output

(No execution)

***

### 🧠 Concept

* Inventory = list of managed nodes

***

## 🧩 8/10 — Create Group in Inventory

### 📘 Topic

Inventory grouping

***

### 🔧 Steps

```bash
vi /home/thor/playbook/inventory-t3q4
```

```ini
[db_servers]
stdb01
```

***

### ✅ Output

(No execution)

***

### 🧠 Concept

* Groups help target multiple servers

***

## 🧩 9/10 — Create File with Content

### 📘 Topic

copy module (`content`)

***

### 🔧 Steps

```bash
vi /home/thor/ansible/playbook-t1q4.yml
```

```yaml
- hosts: localhost
  tasks:
    - copy:
        dest: /tmp/file.txt
        content: "Welcome to the KKE Tests!"
```

Run:

```bash
ansible-playbook -i localhost playbook-t1q4.yml
```

***

### ✅ Output

```bash
changed: [localhost]
```

***

### 🧠 Concept

* `content` → inline text file creation

***

## 🧩 10/10 — Fix Inventory + Remote File Creation

### 📘 Topic

Inventory troubleshooting + file module

***

### 🔧 Steps

#### 1. Fix inventory

```bash
vi /home/thor/ansible/inventory-t1q1
```

```ini
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n
```

***

#### 2. Create playbook

```bash
vi /home/thor/ansible/playbook-t1q1.yml
```

```yaml
- hosts: all
  become: yes
  tasks:
    - file:
        path: /tmp/file-t1q1.txt
        state: touch
```

***

#### 3. Run

```bash
cd /home/thor/ansible
ansible-playbook -i inventory-t1q1 playbook-t1q1.yml
```

***

### ✅ Output

```bash
PLAY [all]

TASK [Create empty file]
changed: [stapp01]

PLAY RECAP
stapp01 : ok=2 changed=1 failed=0
```

***

## 🔥 Final Cheat Sheet

### 📌 Modules

* `copy`
* `file`

### 📌 File states

```yaml
touch   → create
absent  → delete
```

### 📌 Execution

```bash
ansible-playbook -i inventory playbook.yml
```

### 📌 Common errors

* Wrong group ❌
* Wrong path ❌
* Missing playbook ❌
* Permission issue ❌

***

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>
