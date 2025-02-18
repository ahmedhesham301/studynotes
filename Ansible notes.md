# inventory file
[How to build your inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)  
Ansible inventory file structure
```yaml
all:
  hosts:
    web01:
      ansible_host: <ip>
      ansible_user: <username>
      ansible_ssh_private_key_file: path/to/key.pem

  children:
    groupname:
      hosts:
        web01:  
	web02:
```

To create a group of groups, nest another `children:` section within it.
To add variables in a group level add `vars:`

---
# Ad hoc commands
[Introduction to ad hoc commands](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html))
 To run as sudo add `--become` at the end of the command
## To ping
```bash
ansible <target/group/all> -m ping -i inventory.yaml
``` 
## To install a package
```bash
ansible <target> -m yum -a "name=<packageName> state=present" -i inventory.yaml
```
## to start a service and enable it
```bash
ansible <target> -m service -a "name=<service name> state=started enabled=yes" -i inventory.yaml
```

## To copy a file
```bash
ansible <target> -m copy -a "src=<sourcePath> dest=<distPath>" -i inventory.yaml
```
---
# Playbooks
[Ansible playbooks docs](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)
[Index of all Modules](https://docs.ansible.com/ansible/latest/collections/index_module.html)

to run a play book use
```bash
ansible-playbook -i inventory.yaml playbook.yaml
```
to debug add `-v`,`-vv`,`-vvv` the more the Vs the more the details
to dry run(tests the play book) add -C at the end
## playbook structure
```yaml
---

- name: A name for the play
  hosts: <target>
  become: true # to run as sudo
  tasks:
    - name: A name for the task
      command:
        arg1: value
        arg2: value

```
