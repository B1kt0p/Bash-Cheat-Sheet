# Ansible Cheat Sheet

 A cheat sheet for Ansible.

## Ansible ad-hoc

```bash
exe@ansible:$ ansible \ host01 \
-i hosts.ini \
-u user \
-k \
-m ping
SSH password:
host01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
