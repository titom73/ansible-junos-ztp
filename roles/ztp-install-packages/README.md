# 1. ztp-install-packages overview

This role is in charge of installing and enabling services to support Zero Touch Provisionning (ZTP). In this role, the following actions are executed:

- Bootstrap python on the remote server to support apt module
- Update the apt cache_valid_time
- Install `isc-dhcp-server` to activate DHCP service
- Install `vsftpd` to activate FTP service
- Enable anonymous access to FTP service in read-only mode
- Create FTP directory
- Change ownership of the FTP root directory
- Create repository for both software and configuration files
- restart dhcpd and ftpd service

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/all/ztp-variables.yaml](../../all/ztp-variables.yaml)
```
---
ztp:
  setup:
    ftp:
      home_dir:       # Home dir of the FTP server
  path:               # All paths required by ZTP roles
    ftp_conf:         # FTP directory to store configuration
    ftp_root:         # Home dir of the FTP server
    soft:             # FTP directory to store software
    junos_local:      # Location where Junos configurations are stored
```
