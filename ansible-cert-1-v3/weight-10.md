# Weight: 10

The Nautilus DevOps team is running Ansible using a sudo user on the jump host, where the sudo user is configured to prompt for a password for each execution. Currently, playbooks fail when attempting tasks that require superuser privileges, such as package installation. They aim to ensure that Ansible prompts for a sudo password during playbook execution. Consequently, they have outlined the following requirements to resolve this issue:

Please ensure the necessary adjustments are made within the Ansible configuration located at `/home/thor/ansible-t5q4/ansible-t5q4.cfg`. Avoid creating a new configuration file.

`Note:` This is a sample Ansible configuration. If you intend to test an Ansible playbook using this configuration, you may need to explicitly set the `ANSIBLE_CONFIG` variable.

