# Configure Default SSH User for Ansible

The Nautilus DevOps team aims to manage all servers within the stack using Ansible, utilizing a common sudo user across all servers. They plan to use this user for various tasks on each server. While this isn't finalized, they're starting with testing. Ansible is already installed on the `jump host` via yum. Here's the requirement:

On the `jump host`, modify the default configuration of Ansible to enable the use of `james` as the default SSH user for all hosts. Ensure to make changes within Ansible's default configuration without creating a new one.

***

## Configure Default SSH User for Ansible

### 📌 Objective

Configure Ansible on the jump host to use **`james`** as the default SSH user for all managed hosts by modifying the existing default configuration file.

***

### 🖥 Environment

* Jump Host: `jumphost`
* Ansible installed via `yum`
* Default config file: `/etc/ansible/ansible.cfg`
* Required default SSH user: `james`

***

### 🔧 Implementation Steps

#### 1️⃣ Edit the Default Ansible Configuration

Access the Ansible configuration file:

```bash
thor@jumphost ~$ sudo vi /etc/ansible/ansible.cfg
```

You may see the standard sudo security message:

```
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.
```

Enter the sudo password when prompted.

***

#### 2️⃣ Update the `[defaults]` Section

Ensure the configuration includes:

```ini
[defaults]
host_key_checking = False
remote_user = james
```

> ⚠️ Important:\
> Modify the existing configuration file. Do **not** create a new `ansible.cfg`.

Save and exit the editor.

***

### ✅ Verification

Run the following command to verify the change:

```bash
thor@jumphost ~$ ansible-config dump | grep DEFAULT_REMOTE_USER
```

Expected output:

```
DEFAULT_REMOTE_USER(/etc/ansible/ansible.cfg) = james
```

Example session:

```bash
thor@jumphost ~$ ansible-config dump | grep DEFAULT_REMOTE_USER
DEFAULT_REMOTE_USER(/etc/ansible/ansible.cfg) = james
thor@jumphost ~$
```

***

### 🎯 Result

Ansible will now:

* Use **`james`** as the default SSH user
* Apply this setting to all managed hosts
* Override only if explicitly specified in:
  * Inventory files
  * Playbooks
  * CLI (`-u` option)

***

### 📚 Notes

*   The configuration was updated in the default file:

    ```
    /etc/ansible/ansible.cfg
    ```
* This ensures consistency across the environment without introducing custom configuration files.

***

✔ Configuration completed successfully.
