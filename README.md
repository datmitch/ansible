# mysql_ansible-repo
Ansible repository for hi-mysql

## Setup:
```
pip3 install ansible
```
OR
```
yum install ansible
```

## Ansible Components

### Playbook
A playbook is composed of one or more ‘plays’ in an ordered list. Each play executes part of the overall goal of the playbook, running one or more tasks. Each task calls an Ansible module. [Ansible Playbook Doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

### Inventory
The inventory will contain the list of hosts you want Ansible to manage
[Ansible Inventory Doc](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

### Roles
Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users. [Ansible Role Doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)

### Group Vars
Group Vars (variables) allow you to group up managed hosts and set specific variables for that group (please note, we currently don't utilize group_vars but this can be utilized for future improvements) [Ansible Group Vars Doc](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

## Running ansible-playbook

### Syntax, host, and task checks
An ansible playbooks can be executed in a number of ways as a check without actually connecting and making changes to hosts. Here are a few of the commonly used flags that can be appended to an ansible-playbook command to perform these checks:

1. Checking syntax of playbook to ensure there are no configuration errors:
```
--syntax-check
```
2. Check the hosts to be executed against:
```
--list-hosts
```
3. Check the tasks to be executed:
```
--list-tasks
```
4. Perform a "dry-run" without making any configuration changes:
```
--check
```

A full list of available flags can be found in the [ansible-playbook documentation](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html#ansible-playbook).

### Executing a playbook
There are a number of ways to run an ansible playbook. A playbook could potentially include multiple "plays" that can execute a series of tasks and roles against specified hosts. Refer to the Ansible playbook documentation linked above to see all the possibilities that can be done with playbooks. This is helpful for executing multiple roles against multiple servers with one command in a rolling fashion. In our current setup, we have tasks built out in the role with tags associated with each task. In the future, this can be improved by separating the roles, and moving the individual tasks to the playbook as stated to cut down on the number of commands needed to be run. Here are some examples below:

1. Perform reboots on all of the dev slave nodes:
```
ansible-playbook -i inventory/us-1-zone0/dev.yml playbooks/reboots_mysql.yml -l 'slave' --tags reboot_slave -u <user>
```
Unless specified in the task to perform one node at a time, this command will reboot all of the slave servers in dev at once. We use the -i flag to point to the inventory we want the playbook to run against, then include the path to the playbook we want to run ([reboots_mysql playbook](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/playbooks/reboots_mysql.yml) which references the role [reboots_mysql role](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/roles/reboots_mysql/tasks/main.yml)), the -l flag is used to specify the group of hosts to be run against as configured in the [dev inventory file](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/inventory/us-1-zone0/dev.yml#L31-L36), the --tags flag is used the indicate which specific tasks from the role file we want to run, and the -u flag is used to indicate the user that will be connecting to the servers. In the future, we may want to create an ansible user that can connect to each server and have the necessary sudo privileges, but for now this can be done with your personal user and password.

2. Confirm the tasks and hosts to be executed against to check the gluster status:
```
ansible-playbook -i <path to inventory> playbooks/gluster_status.yml --list-hosts --list-tasks
```
You will notice this command does not include the -l flag or the --tags flag. This is because the [gluster_status playbook](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/playbooks/gluster_status.yml) is only capable of being run against the host group named "gluster" as configured in each inventory page for our environments. It will not be capable of running on any other servers unless the hosts value in the playbook is set to "all", or set to a list of different groups separated by colons, for example, webservers:databases:backups. The --tags flag is also unnecessary as there is only one task in the playbook.

3. Start zabbix-agent on all servers from a given inventory page:
```
ansible-playbook -i <path to inventory> playbooks/zabbix_agent.yml --tags zabbix_start -u <user>
```
Since the [zabbix_agent playbook](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/playbooks/zabbix_agent.yml) is configured to run against all servers, the -l flag is unnecessary, but it can be included if you only want to run the task against a single server or group of servers. In the [zabbix_agent role](https://github.cerner.com/ETS-Alpha/mysql_ansible-repo/blob/master/roles/zabbix_agent/tasks/main.yml), we utilize ansible's built in service module. We specify the name of the service, and the state we want it to be in. If zabbix_agent is already running on a server, it will simply be skipped in the execution, and ansible will move on to the next server in the list.

## Additional resources from other teams

1. Ansible repo used by the ETS Forge team: [ETSForge/ansible-playbook-etstools](https://github.cerner.com/ETSForge/ansible-playbook-etstools)
2. Ansible repo used by the ETS Alpha patient portal team: [ETS-Alpha/patient_portal_ansible](https://github.cerner.com/ETS-Alpha/patient_portal_ansible)
