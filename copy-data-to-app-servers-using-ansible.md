# Copy Data to App Servers using Ansible

The Nautilus DevOps team needs to copy data from the jump host to all application servers in Stratos DC using Ansible. Execute the task with the following details: a. Create an inventory file /home/thor/ansible/inventory on jump\_host and add all application servers as managed nodes. b. Create a playbook /home/thor/ansible/playbook.yml on the jump host to copy the /usr/src/itadmin/index.html file to all application servers, placing it at /opt/itadmin. Note: Validation will run the playbook using the command ansible-playbook -i inventory playbook.yml. Ensure the playbook functions properly without any extra arguments.

## Copy Data to Application Servers Using Ansible

### 📌 Objective

The Nautilus DevOps team needs to copy the file:

```
/usr/src/itadmin/index.html
```

From the **jump host** to all **application servers** in Stratos DC using Ansible.

The file must be placed at:

```
/opt/itadmin/index.html
```

***

## 🖥 Infrastructure Details

| Server     | IP            | User   | Purpose        |
| ---------- | ------------- | ------ | -------------- |
| stapp01    | 172.16.238.10 | tony   | Nautilus App 1 |
| stapp02    | 172.16.238.11 | steve  | Nautilus App 2 |
| stapp03    | 172.16.238.12 | banner | Nautilus App 3 |
| jump\_host | Dynamic       | thor   | Jump Server    |

***

## 🚀 Implementation Steps

### Step 1: Create Ansible Directory

Login to jump host as `thor` and create the Ansible working directory:

```bash
thor@jumphost ~$ mkdir -p /home/thor/ansible
```

***

### Step 2: Create Inventory File

Create inventory file:

```bash
thor@jumphost ~$ vi /home/thor/ansible/inventory
```

Add the following content:

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_password=Ir0nM@n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@ ansible_ssh_common_args='-o StrictHostKeyChecking=no'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_password=BigGr33n ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

Save and exit.

***

### Step 3: Create Playbook

Create playbook file:

```bash
thor@jumphost ~$ vi /home/thor/ansible/playbook.yml
```

Add the following content:

```yaml
---
- name: Copy index.html to application servers
  hosts: app_servers
  become: yes
  tasks:
    - name: Copy index.html to /opt/itadmin
      copy:
        src: /usr/src/itadmin/index.html
        dest: /opt/itadmin/index.html
        mode: '0644'
```

Save and exit.

***

### Step 4: Set Proper Ownership

```bash
thor@jumphost ~$ chown -R thor:thor /home/thor/ansible
```

***

### Step 5: Execute Playbook

Navigate to the directory and run:

```bash
thor@jumphost ~$ cd /home/thor/ansible
thor@jumphost ~/ansible$ ansible-playbook -i inventory playbook.yml
```

***

## ✅ Terminal Output

```bash
PLAY [Copy index.html to application servers] *****************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Copy index.html to /opt/itadmin] ************************************************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ****************************************************************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

***

## 🎯 Result

* All application servers were reachable.
*   The file was successfully copied to:

    ```
    /opt/itadmin/index.html
    ```
* No failures or unreachable hosts.
* Playbook executed successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

***

## 📌 Conclusion

The Ansible inventory and playbook were successfully created and executed from the jump host.\
The task meets all validation requirements and is ready for production or CI/CD integration.

<figure><img src=".gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
