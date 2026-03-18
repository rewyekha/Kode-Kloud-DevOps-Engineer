# Weight: 10

The Nautilus DevOps team encountered challenges with the Ansible manager i.e jump host. They identified a lack of logging capabilities within Ansible, hindering their ability to effectively debug issues. Consequently, they formulated the following requirements to address this issue.

Ensure that the appropriate changes are made to the Ansible log path on the jump host so that Ansible utilizes the `/home/thor/ansible-t5q3/ansible-t5q3.log` file for logging. Modify the Ansible configuration located at `/home/thor/ansible-t5q3/ansible-t5q3.cfg`. Please refrain from creating a new configuration file.

`Note:` This is a sample Ansible configuration. If you intend to test an Ansible playbook using this configuration, you may need to explicitly set the `ANSIBLE_CONFIG` variable.





**Requirements:**

* Update the existing Ansible configuration located at `/home/thor/ansible-t5q3/ansible-t5q3.cfg`.
* Ensure Ansible logs all activity to `/home/thor/ansible-t5q3/ansible-t5q3.log`.
* Do **not** create a new configuration file.
* When testing, explicitly set the `ANSIBLE_CONFIG` variable if needed.

***

### Terminal Output Before Fix

```bash
thor@jump-host ~$ ls /home/thor/ansible-t5q3/ansible-t5q3.cfg
/home/thor/ansible-t5q3/ansible-t5q3.cfg
thor@jump-host ~$ vi /home/thor/ansible-t5q3/ansible-t5q3.cfg
thor@jump-host ~$ export ANSIBLE_CONFIG=/home/thor/ansible-t5q3/ansible-t5q3.cfg
thor@jump-host ~$ ansible --version
ERROR: Error reading config file (/home/thor/ansible-t5q3/ansible-t5q3.cfg): While reading from '<string>' [line 36]: option 'log_path' in section 'defaults' already exists
```

After removing the duplicate `log_path`:

```bash
thor@jump-host ~$ export ANSIBLE_CONFIG=/home/thor/ansible-t5q3/ansible-t5q3.cfg
ansible --version
ansible [core 2.14.18]
  config file = /home/thor/ansible-t5q3/ansible-t5q3.cfg
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  python version = 3.9.19
  jinja version = 3.1.2
  libyaml = True
```

Attempting to ping all hosts:

```bash
thor@jump-host ~$ ansible all -m ping
[WARNING]: Unable to parse /home/thor/.ansible/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available
```

After creating the inventory file:

```bash
thor@jump-host ~$ mkdir -p /home/thor/.ansible
thor@jump-host ~$ vi /home/thor/.ansible/hosts
thor@jump-host ~$ export ANSIBLE_CONFIG=/home/thor/ansible-t5q3/ansible-t5q3.cfg
thor@jump-host ~$ ansible all -m ping
[WARNING]: Unhandled error in Python interpreter discovery for host stbkp01 ...
stapp01 | SUCCESS => {"ping": "pong"}
stapp02 | SUCCESS => {"ping": "pong"}
stapp03 | SUCCESS => {"ping": "pong"}
```

Checking the Ansible log:

```bash
thor@jump-host ~$ cat /home/thor/ansible-t5q3/ansible-t5q3.log
2026-03-18 15:29:34,474 | stbkp01 | UNREACHABLE!
2026-03-18 15:29:34,474 | stapp01 | SUCCESS => {"ping": "pong"}
2026-03-18 15:29:34,490 | stapp02 | SUCCESS => {"ping": "pong"}
2026-03-18 15:29:34,490 | stapp03 | SUCCESS => {"ping": "pong"}
```

***

### Solution Steps

1. **Edit the Ansible configuration file**:

```bash
vi /home/thor/ansible-t5q3/ansible-t5q3.cfg
```

Ensure the `[defaults]` section has only **one** `log_path`:

```ini
[defaults]
log_path = /home/thor/ansible-t5q3/ansible-t5q3.log
```

2. **Export the Ansible config**:

```bash
export ANSIBLE_CONFIG=/home/thor/ansible-t5q3/ansible-t5q3.cfg
```

3. **Test Ansible with localhost**:

```bash
ansible localhost -m ping
```

4. **Verify logs are being written**:

```bash
cat /home/thor/ansible-t5q3/ansible-t5q3.log
```

**Expected Outcome:**

* Ping from localhost (or reachable hosts) logs appear in `/home/thor/ansible-t5q3/ansible-t5q3.log`.
* No duplicate `log_path` errors.
* Task requirement for logging is completed, **even if some dynamic hosts are unreachable**.

***

#### Notes

* Unreachable hosts due to dynamic IPs do **not affect task completion**.
* The key goal is to configure logging for debugging on the jump host.
* For unreachable hosts, you can either update `/etc/hosts` or use a dynamic inventory later if needed.
