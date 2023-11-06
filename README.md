# ansible_training
Basic Ansible Learning Project

 **Cenario**
     ![The Cenario](/img/cenario.png)

**Requirements**:

    - Vagrant
    - Virtualbox/VMware
    - WSL / Native Linux / Mac

1. First create the VM's
```console
cd /ubuntu
vagrant up
```

2. Create ssh-key to login on the target-machines
```console
ssh-keygen -t ed25519 -C "Ansible" -f ~/.ssh/ansible
```

3. Edit hosts file of the "ManagerHost" In my case WLS
```
# Ansible
192.168.1.123 node-3
192.168.1.122 node-2
192.168.1.121 node-1
```

4. Install ansible on Controler machine and add ssh on each of the machines
```console
apt install ansible -y &&
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-1 &&
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-2 &&
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-3 &&
```

5. First ANSIBLE command
```console
ansible all --key-file ~/.ssh/ansible -i inventory -m ping -u vagrant
```

```json
node-3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
node-2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
node-1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
6. Ansible gather_facts
```console
ansible all -m gather_facts --limit node-1
```
> This module is automatically called by playbooks to gather useful variables about remote hosts that can be used in playbooks.

7. ansible adhoc commands
> An Ansible ad hoc command uses the /usr/bin/ansible command-line tool to automate a single task on one or more managed nodes.
"Privilege permitions for exec commands on target machines.
Context: For the apt module we're using is possible to look at the official DOC for it
Here is the link: https://docs.ansible.com/ansible/2.9/modules/apt_module.html"

**NOK:**
```console
ansible all -m apt -a update_cache=true
```
```json
node-3 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock /var/lib/apt/lists/lock"
}
node-1 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock /var/lib/apt/lists/lock"
}
node-2 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock /var/lib/apt/lists/lock"
```
>> To work we need to add --become and --ask-become-me-pass
```console
ansible all -m apt -a update_cache=true --become --ask-become-pass
```
```json
BECOME password:
 node-2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "cache_update_time": 1699108086,
    "cache_updated": true,
    "changed": true
}
node-1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "cache_update_time": 1699108087,
    "cache_updated": true,
    "changed": true
}
node-3 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "cache_update_time": 1699108087,
    "cache_updated": true,
    "changed": true
}
```
> The command bellow we can install vim-nox editor, vagrant does not has sudo pass
> So you can use the command without --ask-become-pass.
```console
ansible all -m apt -a name=vim-nox --become
```

>> To upgrade the system 
```console
ansible all -m apt -a "upgrade=dist" --become
```

8. Playbooks
> A playbook is a multiple-machine deployment system, reusable and repeatable
```console
touch install_apache.yml
```
```yaml
---

- hosts: all        # Run in all Hosts on inventary
  become: true      # Run with sudo privileges
  tasks:            # The tasks to run 

  - name: Update Repository Index        # Simple indicative name
    apt:                                 # Module apt
      update_cache: yes                  # update index Cache

  - name: install apache2 package        # Indicative Name
    apt:                                 # Module APT
      name: apache2                      # The module to install
```
```console
ansible-playbook install_apache.yml

PLAY [all] ****************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************
ok: [node-1]
ok: [node-3]
ok: [node-2]

TASK [install apache2 package] ********************************************************************************************
changed: [node-2]
changed: [node-3]
changed: [node-1]

PLAY RECAP ****************************************************************************************************************
node-1                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node-2                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node-3                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### Hosts Groups
9.We can bind hosts by groups, like the example bellow:

```console
[web_servers]
node-1
node-2

[db_servers]
node-3
node-4

[file_servers]
node-5
node-6
```