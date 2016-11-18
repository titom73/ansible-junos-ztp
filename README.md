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
- `ISC-DHCP-Server` is used as part of the DHCP server to provide IP address on the management network.
- `VSFTPd` is used to publish device's configuration and software packages.

All devices names, Ip addresses loopback addresses etc .. are defined in the [inventory file named hosts.ini](hosts.ini).

# 3. Playbooks

Available playbooks are listed below:
- [`playbook-ztp-setup`](playbook-ztp-setup.yml): Setup all softwares on ZTP servers
- [`playbook-ztp-init`](playbook-ztp-init.yml): Init local repository
- [`playbook-ztp-conf-generate`](playbook-ztp-conf-generate.yml): Create configuration for ZTP and store them locally
- [`playbook-ztp-push-data`](playbook-ztp-push-data.yml): Push configuration generated by `playbook-ztp-conf-generate` to remote ztp servers
- [`playbook-ztp-complete`](`playbook-ztp-complete.yml`): Execute all actions in one playbook

## 3.1 Install ZTP software requirements

A playbook to install all software on the remote ZTP server has been created. It will execute bootstraping to configure Python and then install following packages:
- `isc-dhcp-server`
- `vsftpd`

<aside class="notice">
This playbook is for Debian like system and has been validated with Ubuntu 16.04 LTS
</aside>

## 3.2 Create directory structure

A playbook has been created to clean / initiate local directory structure to store all requested files before pushing them to any ZTP server. Playbook is named `playbook-ztp-init.yml`. Actions are listed below:
- Check if `build_dir` exists
- Delete `build_dir` if it exists
- Create `build_dir`
- Create children directory to store softwares and configuration as well

```
ansible-playbook -i hosts.ini playbook-ztp-init.yml

PLAY [Init ZTP directory strucutre to store local files] ***********************

TASK [ztp-init : Print build path version] *************************************
ok: [ansible01] => {
   "msg": "Build directory conf/ztp"
}

TASK [ztp-init : Check if path exists] *****************************************
ok: [ansible01]

TASK [ztp-init : It exists] ****************************************************
ok: [ansible01] => {
   "msg": "Yay, the path exists! will delete it"
}

TASK [ztp-init : Remove build_dir] *********************************************
changed: [ansible01]

TASK [ztp-init : It doesn't exist] *********************************************
skipping: [ansible01]

TASK [ztp-init : create ztp directory] *****************************************
changed: [ansible01]

TASK [ztp-init : build_dir created] ********************************************
ok: [ansible01] => {
   "msg": "build_dir has been initiated correctly"
}

TASK [ztp-init : create ztp directory for configlet] ***************************
changed: [ansible01]

TASK [ztp-init : create ztp directory for softwares] ***************************
changed: [ansible01]

PLAY RECAP *********************************************************************
ansible01                  : ok=8    changed=4    unreachable=0    failed=0
```

## 3.2 Generate
Even without real devices, it's possible to regenerate configurations for all devices using ansible playbooks provided with the project

You can generate configurations by using the following playbook:
```
ansible-playbook -i hosts.ini playbook-ztp-conf-generate.yml
```
> By default, all configurations generated will be stored under the directory `conf/ztp` as defined by `{{build_dir}}` and will replace existing configuration store there

The output below is an example based:
```
ansible-playbook -i hosts.ini playbook-ztp-conf-generate.yml

PLAY [Populate local ZTP configurations] ***************************************

TASK [ztp-create-config : building basic dhcp configuration] *******************
ok: [ztp01]
changed: [srx-02]
ok: [srx-01]
ok: [ansible01]

TASK [ztp-create-config : building ztp configuration for dhcp server] **********
skipping: [ztp01]
skipping: [ansible01]
changed: [srx-02]
changed: [srx-01]

TASK [ztp-create-config : assemble dhcp configuration] *************************
ok: [ztp01]
changed: [srx-02]
ok: [ansible01]
ok: [srx-01]

PLAY RECAP *********************************************************************
ansible01                  : ok=2    changed=0    unreachable=0    failed=0
srx-01                     : ok=3    changed=1    unreachable=0    failed=0
srx-02                     : ok=3    changed=3    unreachable=0    failed=0
ztp01                      : ok=2    changed=0    unreachable=0    failed=0
```

## 3.3. Complete ZTP workflow
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

> By default, all configurations generated will be stored under the directory `conf/ztp` and will replace existing configuration store there

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
├── conf
│   └── ztp
│       ├── configlet
│       │   ├── 01_dhcp.conf
│       │   ├── 02_dhcpd_srx-01.conf
│       │   └── 02_dhcpd_srx-02.conf
│       ├── dhcpd.conf
│       └── softwares
├── CONTRIBUTING.md
├── group_vars
│   ├── all
│   │   ├── vars.yaml
│   │   └── ztp-variables.yaml
│   ├── README.md
│   └── srx
├── hosts.ini
├── LICENSE
├── playbook-ztp-complete.yml
├── playbook-ztp-conf-generate.retry
├── playbook-ztp-conf-generate.yml
├── playbook-ztp-init.yml
├── playbook-ztp-push-data.yml
├── README.md
├── requirements.txt
├── roles
│   ├── ztp-create-config
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yaml
│   │   └── templates
│   │       ├── dhcp.j2
│   │       └── host.j2
│   ├── ztp-init
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yaml
│   │   └── vars
│   │       └── main.yaml
│   ├── ztp-install-packages
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yaml
│   │   └── templates
│   │       └── dhcp.j2
│   └── ztp-push-dhcp
│       ├── README.md
│       ├── tasks
│       │   └── main.yaml
│       └── vars
│           └── main.yaml
└── software
    └── junos-srxsme-15.1X49-D50.3-domestic.tgz
```

# 5. Inventory files

Inventory is stored in the `hosts.ini` file. In order to work, your inventory file must follow structure below:

```
########################
## All Lab devices    ##
########################
[all:children]
ztp-servers
ex
ansible-servers

########################
## ZTP devices        ##
########################

[ztp-servers]
ztp01       ansible_host=10.73.1.196

[ztp-servers:vars]
ansible_connection=ssh
ansible_ssh_user=ansible
ansible_ssh_pass='password123'
ansible_sudo_pass='password123'

[ansible-servers]
ansible01       ansible_host=127.0.0.1
```

Then, as DHCP server will push information on a per host basis, we have to define `mac_address` for each host managed by ZTP. Software package is also required to fullfil dhcpd configuration. So you have to add the following variable for your junos devices:

```
########################
## Junos devices      ##
########################
[ex]
FR-EX2200-110       junos_host=10.73.1.8    loopback_ip=100.0.0.1       mgmt_port=me0   mac_address=2c:6b:f5:3a:1d:7f

[ex:vars]
junos_version= "12.3R3.4"
junos_package= "jinstall-ex-2200-12.3R3.4-domestic.tgz"
```

Structure of this inventory file is based on the syntax used in the [EVPN/VXLAN repository](https://github.com/JNPRAutomate/ansible-junos-evpn-vxlan). It means, you can easily merge both project to setup a complete POC.

# 6. Variables

All variables are stored in the [vars.yaml and ztp-variables.yaml](group_vars/all/). Description is available in this directory.

# Setup Environment

## System Requirements

- To use `ztp-install-packages` role you must use a Debian like system suporting `APT`
- User configured for SSH access must be part of `sudo` group on ztp-servers

### Ansible version:

Minimum version: 2.1.2.0
```
ansible --version
ansible 2.1.2.0
```

### ZTP Server
ZTP server: tested on Ubuntu 14.04 / 16.04 and latest
```
ansible-ztp:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 16.04 LTS
Release:        16.04
Codename:       xenial
```

```
tom@automation-ztp:~/ansible-junos-ztp$ lsb_release  -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 14.04.2 LTS
Release:        14.04
Codename:       trusty
```

## Ansible tips

As Ansible is using askpass to setup `ssh` connection, you have to ensure `hostkey` is already configured on your server. If it is not, you can deactivate hostkey checking by ansible. In your ansible configuration add the following line:

```
cat ~/.ansible.cfg
[defaults]
host_key_checking = False
```
This repository uses its own `ansible.cfg` file to store local parameters. Content of this file is provided below:

```
[defaults]
host_key_checking = False
inventory = hosts.ini
```
As inventory file is set to hosts.ini, there is no need to specify the value if you are using this one.

# Contributing

Please refer to the file [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.
