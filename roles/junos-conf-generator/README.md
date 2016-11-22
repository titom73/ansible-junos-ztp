# 1. junos-conf-generator overview

This role is in charge of creating device configuration to push to your ZTP client by using DHCP and FTP. It relies on `template` module from ansible and create one file per device after collecting information in several files:

- Variables in your inventory file
- Variables in your `group_vars` file
- Variables in your `host_vars` file

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/all/ztp-variables](../../group_vars/all/ztp-variables.yaml)
```
---
- ztp:
    path:
      junos_local: conf/ztp            # Local directory where to store configuration file
```
