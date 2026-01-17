# Ansible Cheat Sheet

! A cheat sheet for Ansible.

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

## Playbook verification

```bash
example: ansible-playbook playbook.yml -i hosts.ini --diff --check
example: ansible-playbook playbook.yml -i hosts.ini --syntax-check

--check—dry-run    — dry-run playbook. Check-mode support required
--syntax-check     - requires module support
--dif              - displays information about the changes made. Check-mode support required
```

## Ansible-lint
! A command-line tool for syntax checking yaml playbook code
! Install: pip installansible-lint | aptinstallansible-lint

```bash
example: ansible-lint --profile basic --nocolor playbook-lint.yml

-L                       — List rule
Tag:skip_ansible_lint    - to ignore a task when checking with a linter

``` 




