# 1. ztp-push-dhcp overview

This role is in charge of updating remote servers with configuration for DHCP server and devices.

- Copy dhcpd.conf from `{{build_dir}}` to `/etc/dhcp/dhcpd.conf`
- Restart dhcp service on the remote server to apply latest configuration
- Copy devices' configuration from `{{ztp.path.junos_local}}` to `{{ztp.path.ftp_root}}/{{ztp.path.ftp_conf}}`
- Copy content of software repository `{{ztp.path.software_local}}` to  `{{ztp.path.ftp_root}}/{{ztp.path.soft}}`

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/all/ztp-variables.yaml](../../group_vars/all/ztp-variables.yaml)
```yaml
---
  ztp:
    path:             # All paths required by ZTP roles
      ftp_conf:       # FTP directory to store configuration
      ftp_root:       # Home dir of the FTP server
      soft:           # FTP directory to store software
      junos_local:    # Location where Junos configurations are stored
      software_local: # Location where software packages are stored locally
```

## 1.2 Extra options

In order to speed up the copy process, you can enable synchronize module based on `rsync`. By default, this role is based on `copy` because there is no dependency with external tool.
To activate synchronize feature, configure `sync` option when calling the role in your playbook.

```
---
  ### Send files to ZTP servers
  - name: Send ZTP configurations & softwares to server
    hosts: ztp-servers
    connection: ssh
    gather_facts: no
    roles:
      - {role: ztp-push-dhcp, become: yes, sync: true}
```

> As `synchronize` module relies on rsync, authentication must be configured out of this playbook. You can use ssh-keys to avoid password prompt or you will have to enter your password when playbook will upload files.
