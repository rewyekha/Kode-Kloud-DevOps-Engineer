# Weight: 10

The Nautilus DevOps team is working to create some data on different app servers in using Ansible. They have some specific requirements related to this task. Find below more details about the same:

a. You can utilise the inventory file `/home/thor/playbook/inventory-t4q2`, present on the `jump host`.

b. Create a playbook named `/home/thor/playbook/playbook-t4q2.yml` to update the permissions of file `/opt/file-t4q2.txt` to `0444` on all app servers.

`Note:` Validation will attempt to execute the playbook using the command `ansible-playbook -i inventory-t4q2 playbook-t4q2.yml`. Please ensure the playbook functions correctly with this command alone, without requiring any additional arguments.

```bash
thor@jump-host ~$ cat /home/thor/playbook/inventory-t4q2
[applications]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner

thor@jump-host ~$ cat /home/thor/playbook/playbook-t4q2.yml
---
- name: Update permissions of /opt/file-t4q2.txt on all application servers
  hosts: applications
  become: yes
  tasks:
    - name: Set file permissions to 0444
      file:
        path: /opt/file-t4q2.txt
        mode: '0444'

thor@jump-host ~$ cd /home/thor/playbook
thor@jump-host ~/playbook$ ansible-playbook -i inventory-t4q2 playbook-t4q2.yml

PLAY [Update permissions of /opt/file-t4q2.txt on all application servers] ******

TASK [Gathering Facts] ***********************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Set file permissions to 0444] *********************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ***********************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0

thor@jump-host ~/playbook$ ansible all -i inventory-t4q2 -m ping
stapp01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
stapp02 | SUCCESS => {
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
