# Ansible Playbooks for RaspberryPi :strawberry:

Welcome to the Ultimate Raspberry Pi Setup repository! :strawberry:

This project is my go-to resource for effortlessly configuring Raspberry Pi in an easy and straightforward way, powered by the magic of Ansible. :tophat: ✨

## Index
1. [Project Structure](#project-structure)
2. [Previous Steps before executing Ansible Playbooks](#previous-steps-before-executing-ansible-playbooks)
3. [Sensitive Data managed by Ansible Vault](#sensitive-data-managed-by-ansible-vault)
4. [To enable PubkeyAuthentication in your Raspberry Pi](#to-enable-pubkeyauthentication-in-your-raspberry-pi)
5. [Launching Base Ansible Playbook](#launching-base-ansible-playbook)
    - [Ensure Base Packages Installed on Raspberry Pi](#ensure-base-packages-installed-on-raspberry-pi)
    - [Configure Useful Topics on Your Favourite Shell](#configure-useful-topics-on-your-favourite-shell)
    - [Configure Vim](#configure-vim)
    - [More](#more)
6. [Launching config_services Playbook](#launching-config_services-playbook)
    - [Ensure Docker Installed, Configured, Enabled, and Started](#ensure-docker-installed-configured-enabled-and-started)
    - [Configure the Best Banner of the World](#configure-the-best-banner-of-the-world)
    - [Install, Configure, and Start Fail2ban](#install-configure-and-start-fail2ban)
    - [More](#more-1)
7. [Next Steps](#next-steps)

## Project Structure
```bash
inventories/ # Folder containing all the servers where ansible will run and its configuration
    └── inventory.ini # Main inventory file
plays/ # Folder containing all the playbooks ro be executed on the hosts, we have one playbook per role
    ├── base.yaml # Playbook which executes the base role (basic configuration for the server)
    └── ...
roles/ # Folder containing all the ansible roles (tasks to be executed on the playbooks)
    ├── base/ # Tasks for basic configuration of the server (packages, pubkeys, etc.)
    │     ├── defaults/main.yaml # Default configuration for the role
    │     ├── tasks/
    │     │     ├── base_packages.yaml # Task to ensure the base packages installed
    │     │     ├── main.yaml # File containing the configuration for all the tasks and how to use them
    │     │     └──  ...
    │     └── ...
    ├── config_services/ # Tasks for services configuration (docker, motd, sshd, etc.)
    └──  ...
.gitignore # File including all the files and folder to not push into git
README.md # Repository documentation
```
## Requirements
### Ansible Collections needed
You might already have installed the following collection not included in `ansible-core`. They should be included in the project [requirements.yaml](requirements.yaml):
- `ansible.posix`
- `community.general`

```bash
ansible-galaxy collection install -r requirements.yaml
```

Check whether it is installed:
```bash
ansible-galaxy collection list
```

### Previous Steps before executing Ansible Playbooks
:one: Make sudo

```bash
sudo su
```

:one: Change the sshd port configuration in order to provide a little more security on ssh connections:

- Edit the sshd service config file:
```bash
vim /etc/ssh/sshd_config
```

- Replace the `Port 22` by `Port XXX`
- Restart the service:
```bash
service sshd restart
```

> [!TIP]
> It can be done automatically, but be carefull with the previous configuration and always check the file content before restart the sshd service:
> ```bash
> sed -i 's/#Port 22/Port XXX/' /etc/ssh/sshd_config
> ```

## Sensitive Data managed by Ansible vault
To store the Ansible Sensitive Data we use [Ansible vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html).

To create the vault.yaml file:
```bash
ansible-vault create vault.yaml
```

It will ask for password, and then **vi** editor will open and we need to fulfill it in yaml format like this:

```yaml
ansible_user: "" # user to connect through ssh
ansible_ssh_pass: "" # no needed when authorized key
ansible_become_pass: "" # no needed when ansible_user in visudo
```

Then, Ansible will create a vault file in the folder you executed the `ansible-vault`: `vault.yaml`

To use the vault variables inside the playbooks, we need to include:

```yaml
vars_files:
 - relative/path/from/playbook/to/vault/file
```

- Create a new file `vault_password.txt` and fulfill it just with the content of the vault password.

- Add to the `ansible-playbook` execution `--vault-password-file=vault_password.txt`:

```bash
ansible-playbook playbook... -i .... --vault-password-file=vault_password.txt
```

> [!NOTE]
> If we want to edit the vault file:
> ```bash
> ansible-vault edit vault.yaml --vault-password-file=vault_password.txt
> ```

## To enable PubkeyAuthentication in your raspberry pi
:one: Configure the [playbooks/base.yaml](playbooks/base.yaml) adding the public key / keys, for example:
```yaml
vars:
  base_authorized_keys:
    - user: jiminy-cricket
      pubkey: "the content of the public key"
```
> [!TIP]
> Key pair can be generated using:
> ```bash
> ssh-keygen -t ed25519 -f ~/.ssh/key_name -C "your_email"
> ```
> For security reasons it is hardly recommended to introduce a passphrase.

> [!CAUTION]
> Be completely sure it is the public key and not the private one, because share your private key can lead to serious security problems. Private keys should never be sent or shared.

:two: Optional (but recomended for more security): disable the password authentication in [playbooks/base.yaml](playbooks/base.yaml):
```yaml
vars:
  base_disable_pass_auth: true # By default is false
```

:three: Launch the playbook (with `--diff` flag to see changes)

Tags: `base-keys-config`
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --tags base-keys-config --diff --check
```

:four: Check your new fancy way of authenticate in your Raspberry Pi!

:five: Now you can remove the `ansible_ssh_pass` from the `vault.yaml` file managed by Ansible:
```bash
ansible-vault edit vault.yaml
```
:six: Start the ssh-agent and add your key
```bash
eval $(ssh-agent -s)
```
```bash
ssh-add ~/.ssh/key_name
```

:seven: Try again to run ansible!

## Launching base ansible playbook
#### Launch the full role
Tags: `base`
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base --check
```

#### Ensure base packages installed on raspberrypi
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base-packages --check
```

#### Configure useful topics on your favourite shell
1. Configure your favorite shell on the playbook the var `base_shell: <your_favourite_shell>` (by default it is `base_shell: ".bashrc"`)
2. Launch the playbook:
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base-shell-config --check
```

#### Configure vim
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base-vim-config --check
```

#### Ensure and configure tmux
Tags: `base-tmux`
```bash
ansible-playbook playbooks/base.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base-tmux --check
```

#### Ensure docker installed, configured, enabled and started
1. Configure on your playbook the var `base_docker_enabled: true`
2. Launch the playbook:
```bash
ansible-playbook playbooks/config_services.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags base-docker --check
```

## Launching config_services ansible playbook
#### Launch the full role
Tags: `config-services`
```bash
ansible-playbook playbooks/config_services -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags config-services --check
```

#### Configure the best banner of the world
Tags: `config-services-banner`
```bash
ansible-playbook playbooks/config_services.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags config-services-banner --check
```

#### Install, configure and start fail2ban:
1. Configure on your playbook the var `fail2ban_enabled: true`
2. Launch the playbook:
Tags: `config-services-fail2ban`
```bash
ansible-playbook playbooks/config_services.yaml -i inventories/inventory.ini --vault-password-file=vault_password.txt --diff --tags config-services-fail2ban --check
```

#### More
To check more available tasks check [roles/config_services/tasks/main.yaml](roles/config_services/tasks/main.yaml)

## Next Steps
| Status | Task |
|----------|----------|
| :white_check_mark: | Config the sshd service to not accept password authentication |
| :hourglass_flowing_sand: | Move the roles to another github projects and import them here with ansible-galaxy |
