# Ansible Archive Module

The Nautilus DevOps team has some data on each app server in `Stratos DC` that they want to copy to a different location. However, they want to create an archive of the data first, then they want to copy the same to a different location on the respective app server. Additionally, there are some specific requirements for each server. Perform the task using Ansible playbook as per requirements mentioned below:

Create a playbook named `playbook.yml` under `/home/thor/ansible` directory on `jump host`, an `inventory` file is already placed under `/home/thor/ansible/` directory on `Jump Server` itself.

1. Create an archive `cluster.tar.gz` (make sure archive format is `tar.gz`) of `/usr/src/sysops/` directory ( present on each app server ) and copy it to `/opt/sysops/` directory on all app servers. The user and group `owner` of archive `cluster.tar.gz` should be `tony` for `App Server 1`, `steve` for `App Server 2` and `banner` for `App Server 3`.

`Note:` Validation will try to run playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure playbook works this way, without passing any extra arguments.



## Ansible Playbook: Archive and Distribute Application Data

### Overview

This document describes the implementation of an Ansible playbook used to create a compressed archive of application data on all application servers in the Stratos Datacenter and store it in a designated directory with host-specific ownership requirements.

The automation ensures consistency, reduces manual effort, and enforces correct file permissions per server.

***

### Objective

Create an Ansible playbook that performs the following tasks on all application servers:

1. Create a compressed archive `cluster.tar.gz` from `/usr/src/sysops/`
2. Store the archive in `/opt/sysops/`
3. Set ownership of the archive based on the target server:
   * stapp01 → tony
   * stapp02 → steve
   * stapp03 → banner

***

### Environment Details

#### Control Node (Jump Host)

* Hostname: jump-host
* User: thor
* Password: mjolnir123

#### Managed Nodes

| Server | Hostname | User   | Password |
| ------ | -------- | ------ | -------- |
| App 1  | stapp01  | tony   | Ir0nM@n  |
| App 2  | stapp02  | steve  | Am3ric@  |
| App 3  | stapp03  | banner | BigGr33n |

***

### Directory Structure

On the jump host:

```bash
/home/thor/ansible/
├── ansible.cfg
├── inventory
└── playbook.yml
```

***

### Inventory Configuration

File: `/home/thor/ansible/inventory`

```ini
stapp01 ansible_host=stapp01 ansible_user=tony ansible_ssh_pass=Ir0nM@n
stapp02 ansible_host=stapp02 ansible_user=steve ansible_ssh_pass=Am3ric@
stapp03 ansible_host=stapp03 ansible_user=banner ansible_ssh_pass=BigGr33n
```

***

### Ansible Configuration

File: `/home/thor/ansible/ansible.cfg`

```ini
[defaults]
host_key_checking = False
```

***

### Playbook Implementation

File: `/home/thor/ansible/playbook.yml`

```yaml
---
- name: Create archive and distribute on app servers
  hosts: all
  become: yes

  vars:
    owner_map:
      stapp01: tony
      stapp02: steve
      stapp03: banner

  tasks:

    - name: Ensure destination directory exists
      file:
        path: /opt/sysops
        state: directory
        mode: '0755'

    - name: Create tar.gz archive of /usr/src/sysops
      archive:
        path: /usr/src/sysops/
        dest: /opt/sysops/cluster.tar.gz
        format: gz

    - name: Set ownership of archive per server
      file:
        path: /opt/sysops/cluster.tar.gz
        owner: "{{ owner_map[inventory_hostname] }}"
        group: "{{ owner_map[inventory_hostname] }}"
```

***

### Execution Steps

#### 1. Navigate to Ansible directory

```bash
cd /home/thor/ansible
ls -l
```

#### Output

```bash
total 8
-rw-r--r-- 1 thor thor  36 Apr 12 08:55 ansible.cfg
-rw-r--r-- 1 thor thor 219 Apr 12 08:55 inventory
```

***

#### 2. Validate Playbook Syntax

```bash
ansible-playbook -i inventory playbook.yml --syntax-check
```

#### Output

```bash
playbook: playbook.yml
```

***

#### 3. Run Ansible Playbook

```bash
ansible-playbook -i inventory playbook.yml
```

#### Output

```bash
PLAY [Create archive and distribute on app servers] ***********************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Ensure destination directory exists] ********************************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Create tar.gz archive of /usr/src/sysops] ***************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Set ownership of archive per server] ****************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ****************************************************************************************************************
stapp01 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

***

### Verification

#### Check on stapp01

```bash
ssh tony@stapp01
ls -l /opt/sysops/
```

#### Output

```bash
tony@stapp01's password:
Last login: Sun Apr 12 09:03:31 2026 from 10.244.81.14

[tony@stapp01 ~]$ ls -l /opt/sysops/
total 4
-rw-r--r-- 1 tony tony 218 Apr 12 09:03 cluster.tar.gz
```

***

#### Check on stapp02

```bash
ssh steve@stapp02
ls -l /opt/sysops/
```

Expected output:

```bash
-rw-r--r-- 1 steve steve cluster.tar.gz
```

***

#### Check on stapp03

```bash
ssh banner@stapp03
ls -l /opt/sysops/
```

Expected output:

```bash
-rw-r--r-- 1 banner banner cluster.tar.gz
```

***

### Conclusion

The Ansible playbook successfully automates the creation of a compressed archive from `/usr/src/sysops/` on all application servers and ensures it is stored in `/opt/sysops/` with correct ownership based on the server identity.

This approach ensures:

* Consistency across environments
* Reduced manual intervention
* Proper access control per server

The implementation is idempotent and can be safely re-executed without unintended changes.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
