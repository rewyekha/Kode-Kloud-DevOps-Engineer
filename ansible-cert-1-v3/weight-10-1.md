# Weight: 10

The Nautilus DevOps team encountered challenges with the Ansible manager i.e jump host. They identified a lack of logging capabilities within Ansible, hindering their ability to effectively debug issues. Consequently, they formulated the following requirements to address this issue.

Ensure that the appropriate changes are made to the Ansible log path on the jump host so that Ansible utilizes the `/home/thor/ansible-t5q3/ansible-t5q3.log` file for logging. Modify the Ansible configuration located at `/home/thor/ansible-t5q3/ansible-t5q3.cfg`. Please refrain from creating a new configuration file.

`Note:` This is a sample Ansible configuration. If you intend to test an Ansible playbook using this configuration, you may need to explicitly set the `ANSIBLE_CONFIG` variable.



