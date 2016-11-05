# GROUP_VARS structure

# ZTP Variables

ZTP roles require some variables to work as expected. All these variables are defined in [ztp-servers/ztp.yaml](ztp-servers/ztp.yaml)

```yaml
---
  ztp:
    setup:               # Group all variable related to the setup (mainly used in ztp-install-packages role)
      dhcp:
        ranges:          # Group RANGE definition
          - range:       # Range configured to the management network
            netmask:     # Netmask
            gw:          # Default Gateway
            pool_low:    # Begining of the IP Scope
            pool_high:   # Begining of the IP Scope
            domain_name: # domain-name pushed by DHCP
    ftp:
      home_dir:          # Home dir of the FTP server

    configuration:
      method:            # Method configured to get both software and configuration
      server:            # All servers configured in dhcpd.conf
        ftp:             # TFTP server in case of this method is used (must be set by any value if not used)
        log:             # Log server
        ntp:             # NTP server

    path:                # All paths required by ZTP roles
      ftp_conf:          # FTP directory to store configuration
      ftp_root:          # Home dir of the FTP server
      soft:              # FTP directory to store software
      junos_local:       # Location where Junos configurations are stored
      software_local:    # Location where software packages are stored locally
```

# Variables for Junos devices

In order to generate correct `dhcpd` configuration, we have to provide package name to retireve with FTP service. These variables are defined in the [`hosts.ini`](../hosts.ini) file

```
junos_version=        # Junos Version (Information only)
junos_package=        # Package name sotred in your repository and pushed in your FTP
```
