# Ansible Proxmox Ubuntu Cloud-Init

This Ansible playbook creates an Ubuntu virtual machine cloud-init template to Proxmox.

## Overview

The playbook performs the following actions:

1.  Downloads an Ubuntu cloud image.
2.  Creates a new template in Proxmox.
3.  Configures cloud-init with user data.

## Requirements

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Proxmox VE](https://www.proxmox.com/en/proxmox-ve)
- The [`community.general`](https://docs.ansible.com/ansible/latest/collections/community/general/) Ansible collection.

## Getting Started

1. **Fork the Repository:**
   - Visit the GitHub repository
   - Click the "Fork" button in the top-right corner
   - Clone your forked repository:
     ```bash
     git clone https://github.com/YOUR_USERNAME/ansible-proxmox-ubuntu-cloudinit.git
     cd ansible-proxmox-ubuntu-cloudinit
     ```

2. **Configure Remote:**
   ```bash
   # Add the upstream repository
   git remote add upstream https://github.com/ORIGINAL_OWNER/ansible-proxmox-ubuntu-cloudinit.git

   # Verify remotes
   git remote -v
   ```

## Usage

1.  **Install required collections:**

    ```bash
    ansible-galaxy collection install -r requirements.yml
    ```

2.  **Install and configure pre-commit:**

    ```bash
    # Install pre-commit
    pip install pre-commit

    # Install the pre-commit hooks
    pre-commit install --install-hooks

    # (Optional) Run pre-commit on all files
    pre-commit run --all-files
    ```

    This will install pre-commit hooks that:
    - Check for trailing whitespace
    - Fix end of files
    - Validate YAML syntax
    - Check for secrets using Gitleaks

3.  **Create `vars/secrets.yml`:**

    ```bash
    ansible-vault create vars/secrets.yml
    ```

    Populate the file with your secrets, based on `vars/secrets_example.yml`.
4. **Customize Variables:**
    Edit `vars/main.yml` and adjust any variables as needed.
5.  **Run the playbook:**

    ```bash
    ansible-playbook main.yml -i hosts.yml --ask-vault-pass
    ```

    You will be prompted for the Ansible Vault password.

## Variables

The playbook uses variables defined in `vars/main.yml` and `vars/secrets.yml`.

### `vars/main.yml`

| Variable             | Description                                                                  | Default Value                                                                |
| -------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `timezone`           | The timezone to set for the virtual machine.                                 | `Pacific/Auckland`                                                           |
| `proxmox.node`       | The Proxmox node to use.                                                     | `pve`                                                                        |
| `proxmox.storage`    | The Proxmox storage to use for the virtual machine's disk.                   | `storage`                                                                    |
| `cloud_image.url`    | The URL of the Ubuntu cloud image.                                           | `https://cloud-images.ubuntu.com/releases/noble/release/ubuntu-24.04-server-cloudimg-amd64.img` |
| `cloud_image.storage`| The Proxmox storage to use for storing the downloaded cloud image.           | `/var/lib/vz/template/iso`                                                   |
| `cloud_image.name`   | The name of the downloaded cloud image.                                      | `ubuntu-24.04-server-cloudimg-amd64.img`                                     |
| `vm.id`              | The ID of the virtual machine.                                               | `9001`                                                                       |
| `vm.cores`           | The number of CPU cores for the virtual machine.                             | `2`                                                                          |
| `vm.cpu`             | The CPU type                                                                | `host`                                                                       |
| `vm.memory`          | The amount of RAM for the virtual machine (in MB).                           | `4096`                                                                       |
| `vm.resize_disk`     | The size to resize the virtual machine's disk to (in GB).                    | `20`                                                                         |
| `vm.QEMU_AGENT`      | Enable QEMU Guest Agent.                                                      | `true`                                                                       |
| `vm.UEFI`            | Enable UEFI boot.                                                             | `true`                                                                       |
| `template.name`      | The name of the cloud-init template.                                         | `ubuntu-24.04-cloudinit-template`                                            |
| `user_data.file`     | The name of the cloud-init user data file.                                   | `user-data.yaml.j2`                                                          |
| `user_data.storage`  | The Proxmox storage to use for storing the cloud-init user data file.        | `local`                                                                      |
| `user_data.storage_path` | The path on the Proxmox storage to store the cloud-init user data file. | `/var/lib/vz/snippets`                                                       |
| `users.name`         | The username for the default user.                                           | `matt`                                                                       |
| `users.sudo`         | Sudo permissions for the user.                                               | `ALL=(ALL) NOPASSWD:ALL`                                                     |
| `users.uid`          | User ID                                                                      | `1027`                                                                       |
| `users.groups`       | User groups                                                                  | `users, admin, docker`                                                       |
| `users.shell`        | User's shell                                                                 | `/bin/bash`                                                                  |
| `chpasswd.expire`    | Whether or not to expire the password.                                       | `False`                                                                                                  |
| `package.upgrade`    | Whether to upgrade packages during provisioning.                             | `true`                                                                       |
| `package.reboot_if_required` | Whether to reboot the VM if required after package upgrades.                | `true`                                                                       |

### `vars/secrets.yml`

This file contains sensitive information and should be encrypted using Ansible Vault. An example file, `vars/secrets_example.yml` is provided.

| Variable      | Description                                      |
| ------------- | ------------------------------------------------ |
| `sshkey`      | The SSH public key to add to the authorized\_keys file for the default user. |
| `cipassword` | The password for the default user.                 |


## Cloud-Init Configuration

The cloud-init configuration is defined in `templates/user-data.yaml.j2`. This template uses variables from `vars/main.yml` and `vars/secrets.yml` to customize the user data. The configuration includes:

-   Setting the timezone.
-   Creating a default user with a specified username, password, SSH key, and sudo privileges.
-   Updating and upgrading packages.
-   Installing Docker.
-   Disabling swap.
-   Enabling and starting the QEMU guest agent.

## Notes

-   The playbook assumes you have a Proxmox cluster set up and configured.
-   The cloud image will be downloaded to the specified Proxmox storage.
-   The virtual machine template will be created with the specified ID, cores, memory, and disk size.
-   The cloud-init user data will be stored in a snippet on the specified Proxmox storage.
