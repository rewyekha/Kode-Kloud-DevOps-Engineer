# Weight: 10

There is data on `jump host` that needs to be copied on `all application servers` in `Stratos DC`. Nautilus DevOps team want to perform this task using `Ansible`. Perform the task as per details mentioned below:

a. On `jump host` we already have inventory file `/home/thor/ansible/inventory-t2q1`.

b. On `jump host` create a playbook `/home/thor/ansible/playbook-t2q1.yml` to copy `/usr/src/itadmin-t2q1/index-t2q1.html` file to all application servers at location `/opt/itadmin-t2q1`.

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory-t2q1 playbook-t2q1.yml` so please make sure the playbook works this way without passing any extra arguments.



```bash
thor@jump-host ~$ cat /home/thor/ansible/inventory-t2q1
[applications]
stapp01 ansible_host=stapp01 ansible_ssh_pass=Ir0nM@n ansible_user=tony
stapp02 ansible_host=stapp02 ansible_ssh_pass=Am3ric@ ansible_user=steve
stapp03 ansible_host=stapp03 ansible_ssh_pass=BigGr33n ansible_user=banner

thor@jump-host ~$ cat /home/thor/ansible/playbook-t2q1.yml
---
- name: Copy HTML file to all application servers
  hosts: applications
  become: yes
  tasks:
    - name: Ensure destination directory exists
      file:
        path: /opt/itadmin-t2q1
        state: directory
        mode: '0755'

    - name: Copy index-t2q1.html to application servers
      copy:
        src: /usr/src/itadmin-t2q1/index-t2q1.html
        dest: /opt/itadmin-t2q1/index-t2q1.html
        owner: root
        group: root
        mode: '0644'

thor@jump-host ~$ cd /home/thor/ansible
thor@jump-host ~/ansible$ ansible-playbook -i inventory-t2q1 playbook-t2q1.yml

PLAY [Copy HTML file to all application servers] ************************************

TASK [Gathering Facts] **************************************************************
ok: [stapp03]
ok: [stapp01]
ok: [stapp02]

TASK [Ensure destination directory exists] ******************************************
ok: [stapp01]
ok: [stapp03]
ok: [stapp02]

TASK [Copy index-t2q1.html to application servers] **********************************
changed: [stapp01]
changed: [stapp03]
changed: [stapp02]

PLAY RECAP **************************************************************************
stapp01                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
