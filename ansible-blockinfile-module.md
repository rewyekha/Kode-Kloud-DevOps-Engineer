# Ansible Blockinfile Module

The Nautilus DevOps team wants to install and set up a simple `httpd` web server on all app servers in `Stratos DC`. Additionally, they want to deploy a sample web page for now using Ansible only. Therefore, write the required playbook to complete this task. Find more details about the task below.

We already have an `inventory` file under `/home/thor/ansible` directory on `jump host`. Create a `playbook.yml` under `/home/thor/ansible` directory on `jump host` itself.

1. Using the playbook, install `httpd` web server on all app servers. Additionally, make sure its service should up and running.
2.  Using `blockinfile` Ansible module add some content in `/var/www/html/index.html` file. Below is the content:

    `Welcome to XfusionCorp!`

    `This is Nautilus sample file, created using Ansible!`

    `Please do not modify this file manually!`
3. The `/var/www/html/index.html` file's user and group `owner` should be `apache` on all app servers.
4. The `/var/www/html/index.html` file's permissions should be `0655` on all app servers.

`Note:`

i. Validation will try to run the playbook using command `ansible-playbook -i inventory playbook.yml` so please make sure the playbook works this way without passing any extra arguments.\
ii. Do not use any custom or empty `marker` for `blockinfile` module.



***

## Deployment of Apache Web Server Using Ansible on Nautilus App Servers

### Overview

The Nautilus DevOps team required automation to install and configure an Apache HTTP server (`httpd`) across all application servers in the Stratos Datacenter using Ansible. The configuration also includes deploying a sample web page with correct permissions and ownership.

This document outlines the complete implementation process executed from the jump host.

***

## Environment Details

| Server Role  | Hostname  | User   | Password   |
| ------------ | --------- | ------ | ---------- |
| App Server 1 | stapp01   | tony   | Ir0nM@n    |
| App Server 2 | stapp02   | steve  | Am3ric@    |
| App Server 3 | stapp03   | banner | BigGr33n   |
| Jump Host    | jump-host | thor   | mjolnir123 |

***

## Objective

Using Ansible:

* Install `httpd` on all application servers
* Ensure the service is started and enabled
* Deploy `/var/www/html/index.html` using `blockinfile`
* Set correct ownership: `apache:apache`
* Set permissions: `0655`

***

## Step 1: Navigate to Ansible Directory

```bash
cd /home/thor/ansible
ls
```

#### Output

```bash
ansible.cfg  inventory
```

***

## Step 2: Verify Inventory File

```bash
cat inventory
```

#### Initial Output (before fix)

```bash
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```

#### Issue Identified

No group defined for application servers, causing Ansible to fail when targeting `hosts: app`.

***

## Step 3: Fix Inventory File

```bash
vi inventory
```

#### Updated Inventory

```ini
[app]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner
```

***

## Step 4: Create Ansible Playbook

```bash
vi playbook.yml
```

#### Playbook Content

```yaml
---
- name: Install and configure httpd on app servers
  hosts: app
  become: yes

  tasks:
    - name: Install httpd package
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd service
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Add content to index.html using blockinfile
      blockinfile:
        path: /var/www/html/index.html
        create: yes
        block: |
          Welcome to XfusionCorp!

          This is  Nautilus sample file, created using Ansible!

          Please do not modify this file manually!

    - name: Set ownership of index.html
      file:
        path: /var/www/html/index.html
        owner: apache
        group: apache

    - name: Set permissions on index.html
      file:
        path: /var/www/html/index.html
        mode: "0655"
```

***

## Step 5: Execute Ansible Playbook

```bash
ansible-playbook -i inventory playbook.yml
```

#### Output

```bash
PLAY [Install and configure httpd on app servers] *****************************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [stapp03]
ok: [stapp02]
ok: [stapp01]

TASK [Install httpd package] ***************************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Start and enable httpd service] ******************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Add content to index.html using blockinfile] *****************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Set ownership of index.html] *********************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Set permissions on index.html] *******************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP *********************************************************************************************************
stapp01 : ok=6 changed=5 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp02 : ok=6 changed=5 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
stapp03 : ok=6 changed=5 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

***

## Step 6: Validation on Application Server

```bash
ssh tony@stapp01
```

#### Service Status Check

```bash
systemctl status httpd
```

#### Output

```bash
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Sun 2026-04-12 09:22:48 UTC
     Main PID: 12310 (httpd)
     Tasks: 177 (limit: 404712)
     Memory: 14.9M
     CPU: 86ms
     CGroup: /system.slice/httpd.service
```

***

#### Verify Web File

```bash
ls -l /var/www/html/index.html
```

#### Output

```bash
-rw-r-xr-x 1 apache apache 179 Apr 12 09:22 /var/www/html/index.html
```

***

#### Verify File Content

```bash
cat /var/www/html/index.html
```

#### Output

```bash
# BEGIN ANSIBLE MANAGED BLOCK
Welcome to XfusionCorp!

This is  Nautilus sample file, created using Ansible!

Please do not modify this file manually!
# END ANSIBLE MANAGED BLOCK
```

***

## Conclusion

The Ansible playbook successfully automated the installation and configuration of Apache HTTP Server across all application servers. The required web page was deployed with correct permissions and ownership, ensuring compliance with Nautilus DevOps standards.

All validation checks passed successfully.

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
