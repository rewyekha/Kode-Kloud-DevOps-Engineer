# Create Ansible Inventory for App Server Testing

The Nautilus DevOps team is testing Ansible playbooks on various servers within their stack. They've placed some playbooks under `/home/thor/playbook/` directory on the `jump host` and now intend to test them on `app server 2` in `Stratos DC`. However, an inventory file needs creation for Ansible to connect to the respective app. Here are the requirements:

a. Create an ini type Ansible inventory file `/home/thor/playbook/inventory` on `jump host`.

b. Include `App Server 2` in this inventory along with necessary variables for proper functionality.

c. Ensure the inventory hostname corresponds to the `server name` as per the wiki, for example `stapp01` for `app server 1` in `Stratos DC`.

`Note:` Validation will execute the playbook using the command `ansible-playbook -i inventory playbook.yml`. Ensure the playbook functions properly without any extra arguments.

***

## Create Ansible Inventory for App Server Testing

### 📌 Objective

Create an **INI-type Ansible inventory file** at:

```
/home/thor/playbook/inventory
```

Include:

* **App Server 2**
* Correct hostname as per wiki → `stapp02`
* Required SSH variables so playbook runs without extra arguments

Validation command:

```bash
ansible-playbook -i inventory playbook.yml
```

***

## 🖥️ Step 1: Login to Jump Host

```bash
ssh thor@jump_host
```

Enter password:

```
mjolnir123
```

***

## 📂 Step 2: Navigate to Playbook Directory

```bash
cd /home/thor/playbook
```

***

## 📝 Step 3: Create Inventory File

Create the inventory file:

```bash
vi inventory
```

Add the following content:

```ini
[app_servers]
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Save and exit.

***

## 📖 Explanation of Variables

| Variable                  | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| `stapp02`                 | Inventory hostname (must match wiki name) |
| `ansible_host`            | Target server IP                          |
| `ansible_user`            | SSH username                              |
| `ansible_password`        | SSH password                              |
| `ansible_ssh_common_args` | Avoid host key prompt during validation   |

***

## 🔍 Step 4: Verify Inventory

Check file contents:

```bash
cat inventory
```

Expected output:

```ini
[app_servers]
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

***

## 🧪 Step 5: Test Connectivity (Optional)

```bash
ansible -i inventory app_servers -m ping
```

Expected output:

```bash
stapp02 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

***

## 🚀 Final Validation

Run:

```bash
ansible-playbook -i inventory playbook.yml
```

It should execute successfully **without requiring extra arguments**.

***

## ✅ Final Result

✔ Inventory file created at correct path\
✔ App Server 2 added with correct hostname\
✔ Required SSH variables configured\
✔ Playbook runs successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

***

🎯 **Task Completed Successfully**

```bash
thor@jumphost ~$ cd /home/thor/playbook
thor@jumphost ~/playbook$ vi inventory
thor@jumphost ~/playbook$ cat inventory
[app_servers]
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'

thor@jumphost ~/playbook$ ansible -i inventory app_servers -m ping
stapp02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
thor@jumphost ~/playbook$ ansible-playbook -i inventory playbook.yml

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [stapp02]

TASK [Install httpd package] ***************************************************
changed: [stapp02]

TASK [Start service httpd] *****************************************************
changed: [stapp02]

PLAY RECAP *********************************************************************
stapp02                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

thor@jumphost ~/playbook$ 

```
