# 1. ztp-push-dhcp overview

This role is in charge of updating remote servers with configuration for DHCP server and devices.

- Copy dhcpd.conf from `{{ztp.path.build}}` to `/etc/dhcp/dhcpd.conf`
- Restart dhcp service on the remote server to apply latest configuration
- copy devices' configuration from `{{ztp.path.junos_local}}` to `{{ztp.path.ftp_root}}/{{ztp.path.ftp_conf}}`

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/ztp-servers/ztp.yaml](../../group_vars/ztp-servers)
```yaml
---
  ztp:
    path:             # All paths required by ZTP roles
      build:          # Local directory where to store configuration file
      ftp_conf:       # FTP directory to store configuration
      ftp_root:       # Home dir of the FTP server
      soft:           # FTP directory to store software
      junos_local:    # Location where Junos configurations are stored
```
