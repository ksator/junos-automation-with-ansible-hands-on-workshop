# About this repo

This project is a hands-on workshop with customers, focusing on Junos automation with Ansible 

This repository has been tested using: 
- Ubuntu 16.04
- Ansible 2.4.3.0  
- the version 1.4.3 of the ```Juniper.junos``` role available on Galaxy.  

#  Looking for more details about Junos automation?    

For more examples regarding Junos automation using Ansible, refer to  
https://github.com/ksator/junos-automation-with-ansible

For more Junos automation solutions, refer to  
https://github.com/ksator?tab=repositories  
https://gitlab.com/users/ksator/projects  
https://gist.github.com/ksator/  

# About Junos automation using Ansible

### About Ansible modules for Junos automation

There are two modules libraries to interact with Junos:
- An Ansible library for Junos built by Juniper
    - These modules are available on [Ansible Galaxy website](https://galaxy.ansible.com/Juniper/junos/)   
- An Ansible library for Junos built by Ansible 
    - Since Ansible version >= 2.1, Ansible natively includes [core modules for Junos](http://docs.ansible.com/ansible/latest/list_of_network_modules.html#junos). 
  - These modules are shipped with Ansible
  - The Junos modules included in Ansible core have names which begin with the prefix **junos_**. 
  
These two sets of modules for Junos automation can coexist on the same Ansible control machine.  
Both of them are used in this repository.  

### Ansible modules for Junos built by Juniper and hosted on Galaxy

- They are hosted on the [Ansible Galaxy website](https://galaxy.ansible.com/Juniper/junos/):      
  - The role is Juniper.junos  
- Here's the [source code](https://github.com/Juniper/ansible-junos-stdlib)  
- Until the version 1.4.3 of the modules included in the Juniper.junos role:
  - Their names begin with the prefix **junos_**. 
  - Here's the [doc for the version 1.4.3](http://junos-ansible-modules.readthedocs.io/en/1.4.3/)
  - To download and install them to the Ansible server, execute the command ```sudo ansible-galaxy install Juniper.junos,1.4.3```
- From version 2 of the modules included in the Juniper.junos role: 
  - Their names begin with the prefix **juniper_junos_**. 
  - Here's the [doc for the last version](http://junos-ansible-modules.readthedocs.io/en/stable/) 
  - To download and install them to the Ansible server, execute the command ```sudo ansible-galaxy install Juniper.junos```

### Ansible modules for Junos built and shipped by Ansible  

- Here's the [documentation](http://docs.ansible.com/ansible/latest/list_of_network_modules.html#junos)   
- The Junos modules included in Ansible core have names which begin with the prefix **junos_**. 
- Here's the [source code](https://github.com/ansible/ansible/tree/devel/lib/ansible/modules/network/junos)    
- Installation: They are shipped with ansible itself (from Ansible 2.1). Ansible 2.1 or above is required.  

# Workshop instructions 

###  Install the requirements

Ask to the instructor if this is not already done.  
if this was not already done previously by the instructor, ssh to the provided Ubuntu 16.04 and run these commands to install yourself the Ansible server and it's requirements
```
sudo -s
apt-get update
apt-get upgrade
apt-get install -y python-dev libxml2-dev python-pip libxslt1-dev build-essential libssl-dev libffi-dev git
pip install junos-eznc jxmlease wget jsnapy ansible==2.4.3.0 requests ipaddress cryptography 
ansible-galaxy install Juniper.junos,1.4.3
```
### Verify some details on the Ansible server:
```
sudo -s
pip list
ansible --version
ansible-galaxy list Juniper.junos
ls /etc/ansible/roles/
```

### Familiarize yourself with the topology:

Here's the [topology file](Connection%20Diagram.pdf)  
Ask to the instructor for the devices username and password.  
Run some tests (from the Ansible server, ping and ssh the Junos devices)

### From the Ansible server, clone the repository: 
```
git clone https://github.com/ksator/junos-automation-with-ansible-hands-on-workshop.git
cd junos-automation-with-ansible-hands-on-workshop
ls -l
pwd
```
### Update some files 

Edit the Inventory file to add all the Junos devices refered in [topology file](Connection%20Diagram.pdf) 
```
vi hosts
```
Edit this file to add the devices credentials 
```
vi group_vars/lab/credentials.yml
```

### Check if some ports (ssh, netconf, ...) are reachables from ansible server to all junos devices
```
more wait_for/pb.yml
ansible-playbook wait_for/pb.yml
```

### Get junos facts from all junos devices 
This validates more details (i.e NetConf on the devices, Juniper.junos role on the Ansible server, ...)

```
more junos_get_facts/pb.yml
ansible-playbook junos_get_facts/pb.yml
ansible-playbook junos_get_facts/pb.yml --limit EX4300-15
```
Verify
```
ls junos_get_facts/inventory/
```

### Collect junos configuration files in all format (set, json, xml, text) from all junos devices 
This playbook collects junos configuration files in set/json/text formats from all junos devices 
```
more junos_facts/pb_collect_configuration.yml
ansible-playbook junos_facts/pb_collect_configuration.yml
```
Verify
```
ls junos_facts/configuration/
ls junos_facts/configuration/set/
ls junos_facts/configuration/text/
```
Update the playbook to collect also junos configuration files in xml formats from all junos devices 
```
vi junos_facts/pb_collect_configuration.yml
```
```
ansible-playbook junos_facts/pb_collect_configuration.yml
```
Verify
```
ls junos_facts/configuration/xml
```

### Collect some junos show commands from all junos devices 
Update the show commands list you want to collect
```
vi junos_cli/cli.yml
```
Run this playbook
```
more junos_cli/pb_collect_cli_output.yml
ansible-playbook junos_cli/pb_collect_cli_output.yml
```
Verify
```
ls junos_cli/cli/MX240-10
ls junos_cli/cli/
```

### configure REST API service on your Junos device (ONLY on your device)
Ask to the instructor which device is yours.  
Update the argument ```hosts``` to target only your device. 
```
vi junos_config/pb_lines.yml
```
Execute the playbook in dry run: 
```
ansible-playbook junos_config/pb_lines.yml --check 
```
Execute the playbook in run a dry run with diff:   
``` 
ansible-playbook junos_config/pb_lines.yml --check --diff
```
Execute the playbook:
``` 
ansible-playbook junos_config/pb_lines.yml
```
Update the playbook ```wait_for/pb.yml``` to check if ansible can now reach the port 3000 (REST) on your Junos device:   
```
vi wait_for/pb.yml
```
```
ansible-playbook wait_for/pb.yml
```

### make REST calls from Ansible

ssh to a Junos device and the get corresponding rpc of a show command. Examples:  
```
ksator@mx480-3> show version | display xml rpc
<rpc-reply xmlns:junos="http://xml.juniper.net/junos/17.4R1/junos">
    <rpc>
        <get-software-information>
        </get-software-information>
    </rpc>
    <cli>
        <banner></banner>
    </cli>
</rpc-reply>
```
```
ksator@mx480-3> show bgp neighbor | display xml rpc
<rpc-reply xmlns:junos="http://xml.juniper.net/junos/17.4R1/junos">
    <rpc>
        <get-bgp-neighbor-information>
        </get-bgp-neighbor-information>
    </rpc>
    <cli>
        <banner></banner>
    </cli>
</rpc-reply>
```
Update the playbook pb_rest_calls.yml with to make some REST calls to Junos devices 
```
vi uri/pb_rest_calls.yml
```
```
ansible-playbook uri/pb_rest_calls.yml
```
Verify
```
ls uri/ -l
```
### Generate configuration files from a template and load it to a device
Here's the template
```
more junos_config/templates/replacevlans.j2
```
Define the desired vlans
```
vi junos_config/vlans.yml
```
Ask to the instructor which device is yours.  
Update the argument ```hosts``` to target only your device. 
```
vi junos_config/pb_replace_vlans.yml
```
Enforce the desired state ONLY on your device  
```
ansible-playbook junos_config/pb_replace_vlans.yml
```
ssh to your Junos device and run these commands
``` 
show system commit
show configuration vlans
show configuration vlans | compare rollback 1
show vlans
show vlans | display xml
```
ssh to your Junos device and add a new vlan and remove an existing vlan.  
Then from the ansible server, re-enforce the desired vlans:   
```
ansible-playbook junos_config/pb_replace_vlans.yml
```


### Audit operational states
Update devices specific variables (host_vars directory) to define the physical topology. 
```
ls host_vars
```
Here's an example. For each device: 
- define the list of interfaces that are supposed to be up.  
- For the management interface, add the name of the expected lldp neighbor
```
# more host_vars/MX480-21/topology.yml
neighbors:
   - interface: xe-1/0/0
   - interface: xe-1/2/0
   - interface: fxp0
     name: mgmt-24
```
In order to get the name of the expected lldp neighbor, ssh to the device and run "show lldp neighbors"
```
ksator@mx480-21> show lldp neighbors
Local Interface    Parent Interface    Chassis Id          Port info          System Name
fxp0               -                   2c:6b:f5:9f:6e:c0   581                mgmt-24
```
```
ksator@mx480-21> show lldp neighbors | display xml
<rpc-reply xmlns:junos="http://xml.juniper.net/junos/17.4R1/junos">
    <lldp-neighbors-information junos:style="brief">
        <lldp-neighbor-information>
            <lldp-local-port-id>fxp0</lldp-local-port-id>
            <lldp-local-parent-interface-name>-</lldp-local-parent-interface-name>
            <lldp-remote-chassis-id-subtype>Mac address</lldp-remote-chassis-id-subtype>
            <lldp-remote-chassis-id>2c:6b:f5:9f:6e:c0</lldp-remote-chassis-id>
            <lldp-remote-port-id-subtype>Locally assigned</lldp-remote-port-id-subtype>
            <lldp-remote-port-id>581</lldp-remote-port-id>
            <lldp-remote-system-name>mgmt-24</lldp-remote-system-name>
        </lldp-neighbor-information>
    </lldp-neighbors-information>
    <cli>
        <banner></banner>
    </cli>
</rpc-reply>
```
```
more junos_command/pb_check_op_states.yml
```
Audit the physical topology of your network:  
```
ansible-playbook junos_command/pb_check_op_states.yml
```
The task ```check if interfaces op status is up``` is failing because the [waitfor](junos_command/pb_check_op_states.yml#L36) argument is not parsing the junos output correcltly.  

ssh to the device and run "show xxx | display xml" to understand how to fix this parsing error. Example:  
```
ksator@mx480-21> show interfaces terse xe-1/0/0 | display xml
```
and then fix the playbook 
```
vi junos_command/pb_check_op_states.yml
```
and re-run it to audit the physical topology of your network:  
```
ansible-playbook junos_command/pb_check_op_states.yml
```
