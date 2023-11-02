# ansible_training
Basic Ansible Learning Project

## First create the VM's
````
cd /ubuntu
vagrant up
````

## Create ssh-key to login on the target-machines
````
ssh-keygen -t ed25519 -C "Ansible"
````

## Edit hosts file of the "ManagerHost" In my case WLS
```
# Ansible
192.168.1.123 node-3
192.168.1.122 node-2
192.168.1.121 node-1
```

## Install on each of the machines
```
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-1
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-2
ssh-copy-id -i ~/.ssh/ansible.pub vagrant@node-3
```

## First ANSIBLE command
´ansible all --key-file ~/.ssh/ansible -i inventory -m ping -u vagrant´
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