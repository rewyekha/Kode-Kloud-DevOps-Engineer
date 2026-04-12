# Troubleshoot and Create Ansible Playbook

An Ansible playbook needs completion on the `jump host`, where a team member left off. Below are the details:

1. The inventory file `/home/thor/ansible/inventory` requires adjustments. The playbook must run on `App Server 3` in `Stratos DC`. Update the inventory accordingly.
2. Create a playbook `/home/thor/ansible/playbook.yml`. Include a task to create an empty file `/tmp/file.txt` on `App Server 3`.

`Note:` Validation will run the playbook using the command `ansible-playbook -i inventory playbook.yml`. Ensure the playbook works without any additional arguments.

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Ansible Playbook Completion – App Server 3

### Objective

Complete an Ansible setup on the Jump Host:

* Update the inventory file to target **App Server 3 (stapp03)**.
* Create a playbook that generates an empty file `/tmp/file.txt`.
* Ensure the playbook runs successfully using:

```bash
ansible-playbook -i inventory playbook.yml
```

***

## Step 1: Navigate to Ansible Directory

```bash
cd /home/thor/ansible
ls
```

#### Output

```bash
inventory
```

***

## Step 2: Update Inventory File

Overwrite the inventory file with the correct host details:

```bash
cat <<EOF > inventory
[stratos_dc]
stapp03.stratos.xfusioncorp.com ansible_user=banner ansible_password=BigGr33n
EOF
```

Verify the file:

```bash
vi inventory
```

***

## Step 3: Create the Playbook

Create `playbook.yml`:

```bash
cat <<EOF > playbook.yml
---
- hosts: stratos_dc
  tasks:
    - name: Create an empty file /tmp/file.txt
      file:
        path: /tmp/file.txt
        state: touch
EOF
```

***

## Step 4: Execute the Playbook

Run the validation command:

```bash
ansible-playbook -i inventory playbook.yml
```

***

## Execution Output

```bash
PLAY [stratos_dc] *****************************************************************************

TASK [Gathering Facts] ************************************************************************
ok: [stapp03.stratos.xfusioncorp.com]

TASK [Create an empty file /tmp/file.txt] *****************************************************
changed: [stapp03.stratos.xfusioncorp.com]

PLAY RECAP ************************************************************************************
stapp03.stratos.xfusioncorp.com : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

***

## Result

* Inventory correctly targets **App Server 3**
* Playbook successfully creates `/tmp/file.txt`
* No additional arguments required
* Validation command works as expected
