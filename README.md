ACL flow-based monitor role
===========================

This role facilitates configuring ACL flow-based monitoring attributes. Flow-based mirroring is a mirroring session in which traffic matches specified policies that are mirrored to a destination port. Port-based mirroring maintains a database that contains all monitoring sessions (including port monitor sessions).
This role is abstracted for dellos10.

The ACL flow-based role requires an SSH connection for connectivity to a Dell EMC Networking device. You can use any of the built-in Dell EMC Networking OS connection variables, or the *provider* dictionary.

Installation
------------

    ansible-galaxy install Dell-Networking.dellos-flow-monitor

Role variables
--------------

- Role is abstracted using the *ansible_net_os_name* variable that can take the dellos10 value
- If variable *dellos_cfg_generate* is set to true, the variable generates the role configuration commands in a file
- Any role variable with a corresponding state variable set to absent negates the configuration of that variable 
- Setting an empty value for any variable negates the corresponding configuration
- *dellos_flow_monitor* (dictionary) with session ID key (in *session <ID>* format) range is 1 to 18
- Variables and values are case-sensitive

**session ID keys**

| Key        | Type                      | Description                                             | Support               |
|------------|---------------------------|---------------------------------------------------------|-----------------------|
| ``session_type``  | string: local_*_,rspan-source,erspan-source      | Configures the monitoring session type            | dellos10 |
| ``description`` | string | Configures the monitor session description | dellos10 |
| ``port_match`` | list | List of interfaces with location source and destination | dellos10 |
| ``port_match.interface_name``     | string | Configures the interface | dellos10 |
| ``port_match.location`` | string: source, destination | Configures the source/destination of interface | dellos10 |
| ``port_match.state``  | string: absent,present\*           | Deletes the interface if set to absent | dellos10 |
| ``flow_based`` | boolean | Enables flow-based monitoring | dellos10 |
| ``shutdown`` | string: up,down\* | Enable/disables the monitoring session | dellos10 |
| ``state``  | string: absent,present\*           | Deletes the monitoring session corresponding to the session ID if set to absent | dellos10 |
                                                                                                      
> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Connection variables
--------------------

Ansible Dell EMC Networking roles require connection information to establish communication with the nodes in your inventory. This information can exist in the Ansible *group_vars* or *host_vars* directories or in the playbook itself.

| Key         | Required | Choices    | Description                                         |
|-------------|----------|------------|-----------------------------------------------------|
| ``host`` | yes      |            | Specifies the hostname or address for connecting to the remote device over the specified transport. |
| ``port`` | no       |            | Specifies the port used to build the connection to the remote device; if unspecified, the value defaults to 22 |
| ``username`` | no       |            | Specifies the username that authenticates the CLI login for the connection to the remote device; if value is unspecified, the ANSIBLE_NET_USERNAME environment variable value is used |
| ``password`` | no       |            | Specifies the password that authenticates the connection to the remote device; if value is unspecified, the ANSIBLE_NET_PASSWORD environment variable value is used |
| ``authorize`` | no       | yes, no\*   | Instructs the module to enter privileged mode on the remote device before sending any commands; if value is unspecified, the ANSIBLE_NET_AUTHORIZE environment variable value is used and the device attempts to execute all commands in non-privileged mode. This key is supported only in dellos9 and dellos6. |
| ``auth_pass`` | no       |            | Specifies the password to use if required to enter privileged mode on the remote device; if *authorize* is set to no, this key is not applicable; if value is unspecified, the ANSIBLE_NET_AUTH_PASS environment variable value is used . This key is supported only in dellos9 and dellos6. |
| ``provider`` | no       |            | Passes all connection arguments as a dictonary object; all constraints (such as required or choices) must be met either by individual arguments or values in this dictonary |

> **NOTE**: Asterisk (\*) denotes the default value if none is specified.

Dependencies
------------

The *dellos-flow-monitor* role is built on modules included in the core Ansible code. These modules were added in Ansible version 2.3.0.

Example playbook
----------------

This example uses the *dellos-flow-monitor* role to configure session monitor configuration. It creates a *hosts* file with the switch details and corresponding variables. The hosts file defines the *ansible_net_os_name* variable with corresponding Dell EMC networking OS name. When *dellos_cfg_generate* is set to true, the variable generates the configuration commands as a .part file in the *build_dir* path. By default it is set to false.
It writes a simple playbook that only references the *dellos-flow-monitor* role.

**Sample hosts file**

    leaf1 ansible_host= <ip_address> ansible_net_os_name= <OS name(dellos10)>

**Sample host_vars/leaf1**

    hostname: leaf1
    provider:
      host: "{{ hostname }}"
      username: xxxxx
      password: xxxxx
    build_dir: ../temp/dellos10
    dellos_flow_monitor:
      session 1:
        session_type: local
        description: "Discription goes here"
        port_match:
          - interface_name: ethernet 1/1/4
            location: source
            state: present
          - interface_name: ethernet 1/1/3
            location: destination
            state: present
        flow_based: true
        shutdown: up
        state: present
      session 2:
        session_type: local
        description: "Discription of session goes here"
        port_match:
          - interface_name: ethernet 1/1/6
            location: source
            state: present
          - interface_name: ethernet 1/1/7
            location: destination
            state: present
        flow_based: true
        shutdown: up
        state: present
    dellos_acl:
	  - name: testflow
	    type: ipv4
            description: testflow description
	    extended: true
            entries:
                - number: 5
                    permit: true
                    protocol: icmp
                    source: any
                    destination: any
                    other_options: capture session 1 count
                    state: present
                - number: 10
                    permit: true
                    protocol: ip
                    source: 102.1.1.0/24
                    destination: any
                    other_option: capture session 1 count byte
                    state: present
                - number: 15
                    permit: false
                    protocol: udp
                    source: any
                    destination: any
                    other_options: capture session 2 count byte
                    state: present
                - number: 20
                    permit: false
                    protocol: tcp
                    source: any
                    destination: any
                    other_options: capture session 2 count byte
                    state: present
                stage_ingress:
                - name: ethernet 1/1/1
                    state: present

**Simple playbook to setup system - leaf.yaml**

    - hosts: leaf1
      roles:
         - Dell-Networking.dellos-flow-monitor
         - Dell-Networking.dellos-acl
                
**Run**

    ansible-playbook -i hosts leaf.yaml

(c) 2017 Dell EMC
