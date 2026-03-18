# Weight: 10

There is data on `jump host` that needs to be copied on `all application servers` in `Stratos DC`. Nautilus DevOps team want to perform this task using `Ansible`. Perform the task as per details mentioned below:

a. On `jump host` we already have inventory file `/home/thor/ansible/inventory-t2q1`.

b. On `jump host` create a playbook `/home/thor/ansible/playbook-t2q1.yml` to copy `/usr/src/security-t2q1/index-t2q1.html` file to all application servers at location `/opt/security-t2q1`.

`Note:` Validation will try to run the playbook using command `ansible-playbook -i inventory-t2q1 playbook-t2q1.yml` so please make sure the playbook works this way without passing any extra arguments.



