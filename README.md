[![Build Status](https://travis-ci.org/titom73/ansible-junos-ztp.svg?branch=master)](https://travis-ci.org/titom73/ansible-junos-ztp)

# Ansible to configure and update ZTP server

Sample project using Ansible to setup and manage ZTP server.

In this project you'll find:
- (1) **A role to install all software on the remote server** with Playbooks and variables based on Ubuntu/Debian system.
- (2) Severals **ansible roles** packaged and documented into [Ansible roles](roles) to configure DHCP server and to deploy configuration files on all remote ZTP servers (FTP).
- (3) **[Examples of ZTP configurations](config/ztp)** files.
- (4) **Playbook to play with ZTP roles** and update ZTP in a more complex project.

## Info on ZTP server

A Complete description of Zero Touch Provisionning (ZTP) is available on Juniper.net
http://www.juniper.net/techpubs/en_US/junos15.1/topics/concept/software-image-and-configuration-automatic-provisioning-understanding.html

# 1. Ansible project to manage ZTP server

This project is managing the creation of a ZTP server running on a Debian like OS
- ISC-DHCP-Server is used as part of the DHCP server to provide IP address on the management network.
- VSFTPd is used to publish device's configuration and software packages.

All devices names, Ip addresses loopback addresses etc .. are defined in the [inventory file named hosts.ini](hosts.ini).

# 2. Generate configurations

Even without real devices, it's possible to regenerate configurations for all devices using ansible playbooks provided with the project

You can generate configurations by using the following playbook:
```
ansible-playbook -i hosts.ini playbook-ztp-conf-generate.yml
```
> By default, all configurations generated will be stored under the directory `config/ztp` and will replace existing configuration store there

The output below is an example based:
```
ansible-playbook -i hosts.ini playbook-ztp-conf-generate.yml

PLAY [Init ZTP directory strucutre to store local files] ***********************

TASK [ztp-init : remove ztp directory] *****************************************
changed: [ztp01]

TASK [ztp-init : create ztp directory] *****************************************
changed: [ztp01]

TASK [ztp-init : create ztp directory for configlet] ***************************
changed: [ztp01]

PLAY [Populate local ZTP configurations] ***************************************

TASK [ztp-create-config : building basic dhcp configuration] *******************
changed: [ztp01]
ok: [srx-02]
ok: [srx-01]

TASK [ztp-create-config : building ztp configuration for dhcp server] **********
skipping: [ztp01]
changed: [srx-01]
changed: [srx-02]

TASK [ztp-create-config : assemble dhcp configuration] *************************
changed: [ztp01]
changed: [srx-02]
changed: [srx-01]

PLAY RECAP *********************************************************************
srx-01                     : ok=3    changed=2    unreachable=0    failed=0
srx-02                     : ok=3    changed=2    unreachable=0    failed=0
ztp01                      : ok=5    changed=5    unreachable=0    failed=0
```

# 3. Complete ZTP workflow
A complete ZTP workflow is providing in the playbook named `playbook-ztp.yml`. This playbook executes all the following actions:

- Install dhcp and ftp servers on the remote servers
- Configure FTP server
- Generate DHCP configuration with all global parameters
- Generate DHCP configuration for all hosts defined in your inventory
- Push DHCP configuration and reload the service
- Push devices' configuration within the FTP server

You can execute this playbook by using the following playbook:
```
ansible-playbook -i hosts.ini playbook-ztp.yml
```

> By default, all configurations generated will be stored under the directory `config/ztp` and will replace existing configuration store there

# 4. Repository strucutre

Repository is using the following structure:

- ***[config/ztp](config/ztp)***: directory to store configuration files generated by ansible
- ***[group_vars/ztp-servers](group_vars/ztp-servers)***: directory to store variables required to run ZTP roles
- ***[roles/ztp-install-packages](roles/ztp-install-packages)***: role to install all packages on remote servers
- ***[roles/ztp-init](roles/ztp-init)***: role to init local directory before storing configuration files
- ***[roles/ztp-create-config](roles/ztp-create-config)***: role to generate configuration for dhcpd
- ***[roles/ztp-push-dhcp](roles/ztp-push-dhcp)***: role to update remote servers with dhcpd configuration and devices' configuration

```
ansible-junos-ztp
├── config
│   ├── srx-01.conf
│   ├── srx-02.conf
│   └── ztp
│       ├── configlet
│       │   ├── 01_dhcp.conf
│       │   ├── 02_dhcpd_srx-01.conf
│       │   └── 02_dhcpd_srx-02.conf
│       └── dhcpd.conf
├── group_vars
│   └── ztp-servers
│       └── ztp.yaml
├── hosts.ini
├── LICENSE
├── playbook-ztp-conf-generate.yml
├── playbook-ztp.yml
├── README.md
└── roles
    ├── ztp-create-config
    │   ├── tasks
    │   │   └── main.yaml
    │   └── templates
    │       ├── dhcp.j2
    │       └── host.j2
    ├── ztp-init
    │   ├── tasks
    │   │   └── main.yaml
    │   ├── templates
    │   └── vars
    │       └── main.yaml
    ├── ztp-install-packages
    │   ├── tasks
    │   │   └── main.yaml
    │   └── templates
    │       └── dhcp.j2
    └── ztp-push-dhcp
        ├── tasks
        │   └── main.yaml
        ├── templates
        └── vars
            └── main.yaml
```

# 5. Inventory files

Inventory is stored in the `hosts.ini` file. In order to work, your inventory file must follow structure below:

```
########################
## ZTP devices        ##
########################
[ztp-servers]
ztp01       ansible_host=<IP address of ZTP server>

[ztp-servers:vars]
ansible_connection=ssh
ansible_ssh_user=<remote user>
ansible_ssh_pass=<password to connect with SSH>
ansible_sudo_pass=<password to execute sudo>
```

Then, as DHCP server will push information on a per host basis, we have to define `mac_address` for each host managed by ZTP. Software package is also required to fullfil dhcpd configuration. So you have to add the following variable for your junos devices:

```
[srx]
srx-01       junos_host=0.0.0.0    mac_address=ff:aa:bb:cc:dd:ee
srx-02       junos_host=0.0.0.0    mac_address=aa:bb:cc:dd:ee:ff

[srx:vars]
junos_software = "junos-srxsme-15.1X49-D50.3-domestic.tgz"
```

Structure of this inventory file is based on the syntax used in the [EVPN/VXLAN repository](https://github.com/JNPRAutomate/ansible-junos-evpn-vxlan). It means, you can easily merge both project to setup a complete POC.

# 6. Variables

All variables are stored in the [ztp.yaml](group_vars/). Description is available in this directory.

# Setup Environment

As Ansible is using askpass to setup `ssh` connection, you have to ensure `hostkey` is already configured on your server. If it is not, you can deactivate hostkey checking by ansible. In your ansible configuration add the following line:

```
cat ~/.ansible.cfg
[defaults]
host_key_checking = False
```

# Contributing

Please refer to the file [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

# Requirements
 - Ansible (min 2.0.0)
