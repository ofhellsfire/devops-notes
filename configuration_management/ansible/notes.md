# Ansible Notes

## Documentation

Use `ansible-doc <module name>` to get documentation in the terminal

## Ad-Hoc Commands

If **Ansible** is started without `-m`, then `command` module is used.

##### Execute arbitrary command against inventory

Example:

```
ansible <group_name> -i <inventory_file> -a '<command>'
```

**NOTE**: `-f <num>` option stands for how many forks are created to accomplish the task. It may affect ordering. Use `-f 1` for serial and ordered execution.

Show all information **Ansible** picked about hosts:

```
ansible <group_name> -t <inventory_file> -m setup
```

##### Execute arbitrary command in the background

Example:

```
# Execute job in the background
ansible -i <inventory_file> <group_name> -B <timeout> -P <poll_interval> -a '<command>'

# Check background job
ansible -i <inventory_file> <group_name> -m async_status -a 'jid=<job_id>'
```

##### Execute arbitrary command with pipes

Example:

```
ansible -i <inventory_file> <group_name> -m shell -a 'tail /var/log/syslog | grep something | wc -l'
```

## Inventory

##### Grouping multiple hosts to the one group

Example:

```
[groupA]
10.10.10.10
10.10.10.11

[groupB]
10.10.10.12
10.10.10.13

[multi:children]  # Note: the syntax is [multi_group_name:children]
groupA
groupB

[mutli:vars]  # Allows keeping vars for all hosts
var1=ololo
var2=alala
```

##### Turn off Strict Host Key checking (useful for sandbox and playground envs)

```
[some_group:vars]
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

## Ansible Configuration

1. To force Ansible to reuse existing SSH connection use `pipelining = True` in `ansible.cfg` in `[ssh_connection]` section. It may speed up playbook execution.

## Ansible Playbooks

### Pre Tasks

It's possible to specify `pre_tasks` that will be executed before `tasks`

```
- hosts: myhosts
  pre_tasks:
    - name: Update apt cache if needed
      apt:
        update_cache: true
        cache_valid_time: 3600
```

### Handlers

It's possible to specify `handlers` that can be triggered via `notify`

```
- hosts: myhosts
  handlers:
    - name: restart service
      service:
        name: service
        state: restarted
  tasks:
    - name: change service configuration
      lineinfile: ...
      notify: restart service
```

### Syntax Check

You can check syntax of your playbook from CLI:

```
ansible-playbook -i inventory playbook.yaml --syntax-check
```

## Practical Ideas

1. Setup a job in CI that starts **Ansible** periodically to make sure that configuration has not been changed from outside.

