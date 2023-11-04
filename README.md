# ansible_training
Basic Ansible Learning Project

 **Cenario**
     ![The Cenario](/img/cenario.png)

1. First create the VM's
````
cd /ubuntu
vagrant up
````

2. Create ssh-key to login on the target-machines
````
ssh-keygen -t ed25519 -C "Ansible"
````

3. Edit hosts file of the "ManagerHost" In my case WLS
```
# Ansible
192.168.1.123 node-3
192.168.1.122 node-2
192.168.1.121 node-1
```

4. Install on each of the machines
```
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-1
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-2
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-3
```

5. First ANSIBLE command
> ansible all --key-file ~/.ssh/ansible -i inventory -m ping -u vagrant
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
> ansible all -m gather_facts --limit node-1

7. Privilege permitions for exec commands on target machines.
Context: For the apt module we're using is possible to look at the official DOC for it
Here is the link: https://docs.ansible.com/ansible/2.9/modules/apt_module.html

NOK:
➜  ansible_training git:(main) ✗ ansible all -m apt -a update_cache=true
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
➜  ansible_training git:(main) ✗ ansible all -m apt -a update_cache=true --become --ask-become-pass
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
➜  ansible_training git:(main) ✗ ansible all -m apt -a name=vim-nox --become

>> To upgrade the system 
➜  ansible all -m apt -a "upgrade=dist" --become