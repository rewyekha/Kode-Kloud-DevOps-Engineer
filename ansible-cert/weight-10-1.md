# Weight: 10

Ansible utilizes SSH connections to communicate with remote hosts. The Nautilus DevOps team intends to employ a unified Ansible manager for overseeing several remote hosts. To streamline operations, they seek to implement a common Ansible configuration to govern these hosts.

Create an Ansible configuration file under `/home/thor/ansible-config` directory and disable the `SSH` host key checking for all Ansible managed hosts.



{% hint style="info" %}
Created the Ansible configuration file under "home/thor/ansible-config" directory and disabled the SSH host key checking for Ansible?
{% endhint %}

```bash
thor@jump-host ~$ mkdir -p /home/thor/ansible-config
thor@jump-host ~$ nano /home/thor/ansible-config/ansible.cfg
thor@jump-host ~$ cat /home/thor/ansible-config/ansible.cfg
[defaults]
host_key_checking = False
inventory = /home/thor/ansible-config/inventory
thor@jump-host ~$ export ANSIBLE_CONFIG=/home/thor/ansible-config/ansible.cfg
thor@jump-host ~$ ansible --version
ansible [core 2.15.0]
  config file = /home/thor/ansible-config/ansible.cfg
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.11/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.11.4 (main, Jun 13 2024, 00:00:00) [GCC 12.2.0]
thor@jump-host ~$ ansible all -m ping
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
