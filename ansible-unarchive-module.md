# Ansible Unarchive Module

One of the DevOps team members has created a zip archive on `jump host` in `Stratos DC` that needs to be extracted and copied over to all app servers in `Stratos DC` itself. Because this is a routine task, the `Nautilus` DevOps team has suggested automating it. We can use Ansible since we have been using it for other automation tasks. Below you can find more details about the task:

We have an `inventory` file under `/home/thor/ansible` directory on `jump host`, which should have all the app servers added already.

There is a zip archive `/usr/src/dba/xfusion.zip` on `jump host`.

Create a `playbook.yml` under `/home/thor/ansible/` directory on `jump host` itself to perform the below given tasks.

1. Unzip `/usr/src/dba/xfusion.zip` archive in `/opt/dba/` location on all app servers.
2. Make sure the extracted data must has the respective sudo user as their `user` and `group` owner, i.e tony for app server 1, steve for app server 2, banner for app server 3.
3. The extracted data permissions must be `0777`.

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.



## Ansible Automation: Extract and Distribute Zip Archive to App Servers (Stratos DC)

### 1. Overview

This document describes the automation of a routine DevOps task in the Stratos DC environment using Ansible. The requirement is to extract a zip archive located on the jump host and distribute its contents to all application servers with correct ownership and permissions.

Ansible is used to ensure repeatability, consistency, and automation across all target nodes.

***

### 2. Objective

The playbook must perform the following actions on all application servers:

1. Extract `/usr/src/dba/xfusion.zip` into `/opt/dba/`
2. Set ownership based on the target server:
   * stapp01 → tony
   * stapp02 → steve
   * stapp03 → banner
3. Set permissions of extracted content to `0777`

***

### 3. Environment Details

| Server               | Hostname  | User   |
| -------------------- | --------- | ------ |
| Application Server 1 | stapp01   | tony   |
| Application Server 2 | stapp02   | steve  |
| Application Server 3 | stapp03   | banner |
| Jump Host            | jump-host | thor   |

Ansible control node: jump-host

Inventory file location:

```
/home/thor/ansible/inventory
```

Playbook location:

```
/home/thor/ansible/playbook.yml
```

Archive location (on control node):

```
/usr/src/dba/xfusion.zip
```

***

### 4. Pre-Checks

#### Navigate to Ansible directory

```bash
cd /home/thor/ansible
```

#### Verify inventory file

```bash
ls -l inventory
```

Output:

```
ansible.cfg  inventory
```

#### Verify connectivity to all hosts

```bash
ansible all -i inventory -m ping
```

Output:

```bash
stapp02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

stapp01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

stapp03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

***

### 5. Initial Playbook Creation

#### Create playbook file

```bash
vi playbook.yml
```

Initial incorrect run caused failure due to incorrect unarchive usage:

#### First execution output

```bash
ansible-playbook -i inventory playbook.yml
```

Error:

```bash
TASK [Extract xfusion.zip into /opt/dba] **********************************************************************************
fatal: [stapp01]: FAILED! => {"changed": false, "msg": "Source '/usr/src/dba/xfusion.zip' does not exist"}
fatal: [stapp02]: FAILED! => {"changed": false, "msg": "Source '/usr/src/dba/xfusion.zip' does not exist"}
fatal: [stapp03]: FAILED! => {"changed": false, "msg": "Source '/usr/src/dba/xfusion.zip' does not exist"}
```

#### Root Cause

The playbook incorrectly treated the archive as if it existed on remote servers.

***

### 6. Fix Applied

The issue was resolved by removing:

```
remote_src: yes
```

This ensures Ansible copies the archive from the control node (jump host) to the managed nodes before extraction.

***

### 7. Final Playbook

```yaml
---
- name: Extract and distribute archive to app servers
  hosts: all
  become: yes

  vars:
    owner_map:
      stapp01: tony
      stapp02: steve
      stapp03: banner

  tasks:

    - name: Ensure /opt/dba directory exists
      file:
        path: /opt/dba
        state: directory
        mode: '0777'

    - name: Extract xfusion.zip into /opt/dba
      ansible.builtin.unarchive:
        src: /usr/src/dba/xfusion.zip
        dest: /opt/dba

    - name: Set ownership and permissions
      file:
        path: /opt/dba
        owner: "{{ owner_map[inventory_hostname] }}"
        group: "{{ owner_map[inventory_hostname] }}"
        mode: '0777'
        recurse: yes
```

***

### 8. Final Execution

#### Run playbook

```bash
ansible-playbook -i inventory playbook.yml
```

***

### 9. Final Output

```bash
PLAY [Extract and distribute archive to app servers] **********************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Ensure /opt/dba directory exists] ***********************************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Extract xfusion.zip into /opt/dba] **********************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Set ownership and permissions] **************************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ****************************************************************************************************************
stapp01 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

***

### 10. Validation Summary

* Archive successfully extracted on all application servers
* Correct ownership applied per host
* Permissions set to 0777 recursively
* No task failures observed
* Playbook is idempotent and production-ready

***

### 11. Conclusion

The automation successfully eliminates manual intervention for archive distribution across multiple servers. The Ansible playbook ensures consistency, reduces operational overhead, and can be reused for future deployments with minimal modifications.

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>
