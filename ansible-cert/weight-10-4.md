# Weight: 10

The Nautilus DevOps team is working to create some data on different app servers in using Ansible. They want to create some files/directories and have some specific requirements related to this task. Find below more details about the same:

a. Utilise the inventory file `/home/thor/playbook/inventory-t4q3`, present on the `jump host`.

b. Create a playbook named `/home/thor/playbook/playbook-t4q3.yml` to create a directory named `/opt/backup-t4q3` on `all App Servers`.

`Note:` Validation will attempt to execute the playbook using the command `ansible-playbook -i inventory-t4q3 playbook-t4q3.yml`. Please ensure the playbook functions correctly with this command alone, without requiring any additional arguments.

```bash
thor@jump-host ~$ cat /home/thor/playbook/inventory-t4q3
[applications]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner

thor@jump-host ~$ cat /home/thor/playbook/playbook-t4q3.yml
---
- name: Create backup directory on all application servers
  hosts: applications
  become: yes
  tasks:
    - name: Ensure /opt/backup-t4q3 directory exists
      file:
        path: /opt/backup-t4q3
        state: directory
        mode: '0755'

thor@jump-host ~$ cd /home/thor/playbook
thor@jump-host ~/playbook$ ansible-playbook -i inventory-t4q3 playbook-t4q3.yml

PLAY [Create backup directory on all application servers] ************************

TASK [Gathering Facts] ***********************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Ensure /opt/backup-t4q3 directory exists] **********************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ***********************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0

thor@jump-host ~/playbook$ ansible all -i inventory-t4q3 -m ping
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

