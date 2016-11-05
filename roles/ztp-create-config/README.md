# 1. ztp-create-config overview

This role is in charge of creating DHCP configuration to apply on your ZTP server. It is only an offline role that take all your data and generate configuration file:

- Use `dhcp.j2` to build a generic template for isc-dhcp-server. This template create both a scope and all new fields required for ZTP
- Use `host.j2` to create specific configuration for host where `mac_address` is defined in the inventory
- Assemble all files generated in the previous actions.

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/all/ztp-variables.yaml](../../group_vars/all/ztp-variables.yaml)
```yaml
---
  ztp:
    setup:    #Group all variable related to the setup (mainly used in ztp-install-packages role)
      dhcp:
        range:        # Range configured to the management network
        netmask:      # Netmask
        gw:           # Default Gateway
        pool_low:     # Begining of the IP Scope
        pool_high:    # Begining of the IP Scope
        domain_name:  # domain-name pushed by DHCP

    configuration:
      method:         # Method configured to get both software and configuration
      server:         # All servers configured in dhcpd.conf
        tftp:         # TFTP server in case of this method is used (must be set by any value if not used)
        log:          # Log server
        ntp:          # NTP server

    path:             # All paths required by ZTP roles
      ftp_conf:       # FTP directory to store configuration
      ftp_root:       # Home dir of the FTP server
      soft:           # FTP directory to store software
      junos_local:    # Location where Junos configurations are stored
      software_local: # Location where software packages are stored locally
```

Inventory file must contains the following information for all hosts that need to be defined in your ZTP server:

***[hosts.ini](../../hosts.ini):***
```
[group_Name]
<Your Device Name>       junos_host=<mgt_ip>    mac_address=<device_mac_address>

[group_Name:vars]
junos_versio=<Junos version to use>
junos_package=<filename of the package>
```

## 1.2. DHCP template

```
  #
  # Sample configuration file for ISC dhcpd for Debian
  #
  # Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
  # configuration file instead of this file.
  #

  ddns-update-style none;

  # option definitions common to all supported networks...
  option domain-name "{{ztp.setup.dhcp.domain_name}}";

  default-lease-time 600;
  max-lease-time 7200;

  # If this DHCP server is the official DHCP server for the local
  # network, the authoritative directive should be uncommented.
  authoritative;

  # Use this to send dhcp log messages to a different log file (you also
  # have to hack syslog.conf to complete the redirection).
  log-facility local7;

  # ZTP entries
  option space NEW_OP;
  option NEW_OP.image-file-name code 0 = text;
  option NEW_OP.config-file-name code 1 = text;
  option NEW_OP.image-file-type code 2 = text;
  option NEW_OP.transfer-mode code 3 = text;
  option NEW_OP.alt-image-file-name code 4 = text;
  option NEW_OP-encapsulation code 43 = encapsulate NEW_OP;

  option space ztp-ops;
  option ztp-ops.image-file-name code 0 = text;
  option ztp-ops.config-file-name code 1 = text;
  option ztp-ops.image-file-type code 2 = text;
  option ztp-ops.transfer-mode code 3 = text;
  option ztp-ops-encap code 43 = encapsulate ztp-ops;
  option ztp-ops.ztp-file-server code 150 = { ip-address };

  # Subnet definition
  subnet {{ ztp.setup.dhcp.range}} netmask {{ztp.setup.dhcp.netmask}} {
    range dynamic-bootp {{ztp.setup.dhcp.pool_low}} {{ztp.setup.dhcp.pool_high}};
    option routers {{ztp.setup.dhcp.gateway}};
  }
```

## 1.3. HOST template

```
host {{inventory_hostname}} {
		hardware ethernet {{mac_address}};
		fixed-address {{junos_host}};
		option host-name "{{inventory_hostname}}";
{% if ztp.configuration.server.log is defined %}		option log-servers {{ztp.configuration.server.log}};
{% endif %}
{% if ztp.configuration.server.ntp is defined %}		option ntp-servers {{ztp.configuration.server.ntp}};
{% endif %}
		option ztp-ops.config-file-name "{{ztp.path.ftp_conf}}/{{inventory_hostname}}.config";
		option ztp-ops.ztp-file-server {{ztp.configuration.server.ftp}};
		option ztp-ops.image-file-name "{{ztp.path.soft}}/{{junos_package}}";
  	option ztp-ops.transfer-mode "{{ztp.configuration.method}}";
}
```
