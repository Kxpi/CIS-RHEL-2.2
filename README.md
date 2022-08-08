# CIS-RHEL-2.2 Ansible playbook

 ## Introduction
 This playbook performs chapter 2.2 from CIS RHEL 8 Benchmark - Special Purpose Services. You can read about the tasks needed to be done [here](CIS-RHEL-8-chapter-2.pdf).

## Prerequisites
Assuming you have Python3 installed (my version 3.8.10) and pip, only Ansible is needed.
```
pip install ansible
```
or
```
pip install -r requirements.txt
```
## Usage
There is one playbook with two roles - **audit**, which validates if target node complies with CIS benchmark, and **remediation**, which enforces recommended security policies on target node, making it compliant.
### Default behaviour
```
ansible-playbook cis-rhel-22.yaml -i inv.ini
```
**Note**: using this strategy, audit will gather exact list of packages and non-masked services installed/enabled on target node and it will be automatically used it in remediation role.
### Audit only
```
ansible-playbook cis-rhel-22.yaml -i inv.ini --tags audit
```
### Remediation only
```
ansible-playbook cis-rhel-22.yaml -i inv.ini --tags remediation
```
**Note**: when you run remediation role only, it will work with default values from group_vars - without checking how many of those packages are installed or how many services are enabled and not masked - it will go through the whole list.

## Remediation - soft and hard 
Remediation role can be executed in 2 different ways - soft and hard. Hard remedation uninstalls all unnecessary packages. Soft version also does that but where it is possible, it will mask a service that is allowed or needed to work on target node (for example  because of dependencies). 

Default behavior is soft remediation, but it can be changed to hard remediation by setting **remediation_hard_mode** variable in [custom_vars.yaml](custom_vars.yaml) to **true**:
```
remediation_hard_mode: true
```

This file also contains a variable called **services_without_mask** where you can create custom list of services that should be masked. In this way, when you need certain services to be enabled (but masked) you can specify them in this list. If you need rsync service but don't want to have rpcbind or nfs-server, your list should look like this:
```
services_without_mask:
	- rsync
```
Thanks to that, rsync will stay on the target node as masked service and other ones will be removed.

If you want to use custom services list or perform hard remediation, use **--extra-vars** flag when running playbook and point to file with variables:
```
ansible-playbook cis-rhel-22.yaml -i inv.ini --extra-vars "@custom_vars.yaml"
```