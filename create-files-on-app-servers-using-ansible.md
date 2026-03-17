# Create Files on App Servers using Ansible

The Nautilus DevOps team is testing various Ansible modules on servers in `Stratos DC`. They're currently focusing on file creation on remote hosts using Ansible. Here are the details:

a. Create an inventory file `~/playbook/inventory` on `jump host` and include `all app servers`.\
b. Create a playbook `~/playbook/playbook.yml` to create a blank file `/usr/src/webdata.txt` on `all app servers`.\
c. Set the permissions of the `/usr/src/webdata.txt` file to `0644`.\
d. Ensure the user/group owner of the `/usr/src/webdata.txt` file is `tony` on `app server 1`, `steve` on `app server 2` and `banner` on `app server 3`.\
`Note:` Validation will execute the playbook using the command `ansible-playbook -i inventory playbook.yml`, so ensure the playbook functions correctly without any additional arguments.



***

## Create Files on App Servers using Ansible

### 📌 Objective

The Nautilus DevOps team needs to:

1. Create an inventory file including all app servers
2. Create an Ansible playbook
3. Create a blank file `/usr/src/webdata.txt` on all app servers
4. Set file permissions to `0644`
5. Set ownership as:
   * `tony` on stapp01
   * `steve` on stapp02
   * `banner` on stapp03

Validation command:

```bash
ansible-playbook -i inventory playbook.yml
```

***

## 🏗 Infrastructure Details

| Server  | IP            | User   |
| ------- | ------------- | ------ |
| stapp01 | 172.16.238.10 | tony   |
| stapp02 | 172.16.238.11 | steve  |
| stapp03 | 172.16.238.12 | banner |

***

## ✅ Solution Steps (Executed on jump\_host)

***

### Step 1: Create Playbook Directory

```bash
thor@jumphost ~$ mkdir -p ~/playbook
thor@jumphost ~$ cd ~/playbook
```

***

### Step 2: Create Inventory File

```bash
thor@jumphost ~/playbook$ vi inventory
```

Add the following content:

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Save and exit.

***

### Step 3: Create Playbook File

```bash
thor@jumphost ~/playbook$ vi playbook.yml
```

Add the following content:

```yaml
---
- name: Create webdata file on app servers
  hosts: app_servers
  become: yes

  tasks:
    - name: Create blank file with correct ownership and permissions
      file:
        path: /usr/src/webdata.txt
        state: touch
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0644'
```

Save and exit.

***

### Step 4: Execute the Playbook

```bash
thor@jumphost ~/playbook$ ansible-playbook -i inventory playbook.yml
```

***

## ✅ Successful Output

```bash
PLAY [Create webdata file on app servers] *********************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [stapp02]
ok: [stapp03]
ok: [stapp01]

TASK [Create blank file with correct ownership and permissions] ***********************************************************
changed: [stapp02]
changed: [stapp03]
changed: [stapp01]

PLAY RECAP ****************************************************************************************************************
stapp01 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

***

## 🎯 Result Verification

On each app server:

```bash
ls -l /usr/src/webdata.txt
```

Expected:

```
-rw-r--r-- 1 tony   tony   ... /usr/src/webdata.txt   (stapp01)
-rw-r--r-- 1 steve  steve  ... /usr/src/webdata.txt   (stapp02)
-rw-r--r-- 1 banner banner ... /usr/src/webdata.txt   (stapp03)
```

***

## 🏁 Conclusion

✔ Inventory file created\
✔ Playbook created\
✔ File `/usr/src/webdata.txt` created on all app servers\
✔ Permissions set to `0644`\
✔ Correct ownership applied per server\
✔ Playbook runs successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```
