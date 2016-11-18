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

## 1.2 Variables requested by template

As role is using template to create initial `dhcpd.conf` file, some inputs must be set to match your own environment. These variables are located in [group_vars/all/ztp-variables.yaml](../../all/ztp-variables.yaml)

```
---
  ztp:
    setup:
      dhcp:
        domain_name: lab.jnpr.net
        ranges:
          - range:        # Range definition / Network address
            netmask:      # Network mask
            gateway:      # Default gateway pushed by DHCP
            pool_low:     # First IP address available in the scope
            pool_high:    # Last IP address available in the scope
```

In some cases, you would be interested by adding other paramters such as `name-server`. In this case, you can edit template and variables to add your own constraints.

For more information about how to configure `isc-dhcp-server`, you can refer to its official documentation: [`man 5 dhcpd.conf`](https://www.freebsd.org/cgi/man.cgi?query=dhcpd.conf&sektion=5&apropos=0&manpath=FreeBSD+9.0-RELEASE+and+Ports)
