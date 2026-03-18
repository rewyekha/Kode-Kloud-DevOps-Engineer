# Weight: 10

a. On `jump host` we already have an inventory file `/home/thor/ansible/inventory-t2q2`.

b. On `jump host` create a playbook `/home/thor/ansible/playbook-t2q2.yml` to copy `/usr/src/itadmin-t2q2/system-t2q2.txt` file to all application servers at location `/opt/itadmin-t2q2` with permissions `0600`.

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory-t2q2 playbook-t2q2.yml` so please make sure the playbook works this way without passing any extra arguments.



```bash
thor@jump-host ~/ansible$ cat /home/thor/ansible/inventory-t2q2
[applications]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner

thor@jump-host ~/ansible$ cat /home/thor/ansible/playbook-t2q2.yml
---
- name: Copy system file to all application servers
  hosts: applications
  become: yes
  tasks:
    - name: Ensure destination directory exists
      file:
        path: /opt/itadmin-t2q2
        state: directory
        mode: '0755'

    - name: Copy system-t2q2.txt to application servers
      copy:
        src: /usr/src/itadmin-t2q2/system-t2q2.txt
        dest: /opt/itadmin-t2q2/system-t2q2.txt
        owner: root
        group: root
        mode: '0600'

thor@jump-host ~/ansible$ ansible-playbook -i inventory-t2q2 playbook-t2q2.yml

PLAY [Copy system file to all application servers] *******************************

TASK [Gathering Facts] **********************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Ensure destination directory exists] **************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Copy system-t2q2.txt to application servers] ******************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP **********************************************************************
stapp01                    : ok=3    changed=1    unreachable=0    failed=0
stapp02                    : ok=3    changed=1    unreachable=0    failed=0
stapp03                    : ok=3    changed=1    unreachable=0    failed=0

thor@jump-host ~/ansible$ ansible all -i inventory-t2q2 -m ping
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
