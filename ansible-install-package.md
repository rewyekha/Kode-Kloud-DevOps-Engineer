# Ansible Install Package

The Nautilus Application development team wanted to test some applications on app servers in `Stratos Datacenter`. They shared some pre-requisites with the DevOps team, and packages need to be installed on app servers. Since we are already using Ansible for automating such tasks, please perform this task using Ansible as per details mentioned below:

1. Create an inventory file `/home/thor/playbook/inventory` on `jump host` and add all app servers in it.
2. Create an Ansible playbook `/home/thor/playbook/playbook.yml` to install `logrotate` package on `all app servers` using Ansible `yum` module.
3. Make sure user `thor` should be able to run the playbook on `jump host`.

`Note:` Validation will try to run playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.



## Ansible Automation: Installing logrotate on App Servers (Stratos Datacenter)

### 1. Objective

The Nautilus DevOps team required installation of the `logrotate` package on all application servers using Ansible automation. The task was performed from the jump host (`thor`) using an Ansible playbook and inventory file.

***

### 2. Environment Details

| Server Role          | Hostname  | User   | Purpose              |
| -------------------- | --------- | ------ | -------------------- |
| Application Server 1 | stapp01   | tony   | Nautilus App 1       |
| Application Server 2 | stapp02   | steve  | Nautilus App 2       |
| Application Server 3 | stapp03   | banner | Nautilus App 3       |
| Jump Host            | jump-host | thor   | Ansible control node |

***

### 3. Directory Structure

All Ansible configuration files were created under:

```
/home/thor/playbook
```

***

### 4. Inventory File Configuration

#### File Path

```
/home/thor/playbook/inventory
```

#### Content

```yaml
[app_servers]
stapp01 ansible_user=tony
stapp02 ansible_user=steve
stapp03 ansible_user=banner
```

***

### 5. Playbook Configuration

#### File Path

```
/home/thor/playbook/playbook.yml
```

#### Content

```yaml
---
- name: Install logrotate on app servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Install logrotate package
      yum:
        name: logrotate
        state: present
```

***

### 6. Initial Ansible Version Check

#### Command

```
ansible --version
```

#### Output

```bash
ansible [core 2.14.18]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /home/thor/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.19 (main, Jun 11 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

***

### 7. First Playbook Execution (Failure Due to SSH Authentication)

#### Command

```bash
ansible-playbook -i inventory playbook.yml
```

#### Output

```bash
PLAY [Install logrotate on app servers] ***********************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
fatal: [stapp02]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: steve@stapp02: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}
fatal: [stapp01]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: tony@stapp01: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}
fatal: [stapp03]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: banner@stapp03: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).", "unreachable": true}

PLAY RECAP ****************************************************************************************************************
stapp01 : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=0 changed=0 unreachable=1 failed=0 skipped=0 rescued=0 ignored=0
```

#### Issue Identified

SSH authentication failed due to missing host key verification configuration.

***

### 8. Fix Applied (Host Key Checking Disabled)

#### Commands Executed

```
export ANSIBLE_HOST_KEY_CHECKING=False
```

To make it persistent:

```
echo "export ANSIBLE_HOST_KEY_CHECKING=False" >> ~/.bashrc
source ~/.bashrc
```

***

### 9. Successful Playbook Execution

#### Command

```
ansible-playbook -i inventory playbook.yml
```

#### Output

```bash
PLAY [Install logrotate on app servers] ***********************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [stapp03]
ok: [stapp02]
ok: [stapp01]

TASK [Install logrotate package] ******************************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ****************************************************************************************************************
stapp01 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

***

### 10. Conclusion

The Ansible automation was successfully implemented. The playbook:

* Targeted all application servers via inventory grouping
* Installed `logrotate` using the `yum` module
* Executed successfully after resolving SSH host key verification issues
* Completed without failures across all managed nodes

The task is fully compliant with the requirement that the playbook runs using:

```
ansible-playbook -i inventory playbook.yml
```



<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
