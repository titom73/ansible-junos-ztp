# 1. ztp-create-config overview

This role is in charge of creating DHCP configuration to apply on your ZTP server. It is only an offline role that take all your data and generate configuration file:

- Use `dhcp.j2` to build a generic template for isc-dhcp-server. This template create both a scope and all new fields required for ZTP
- Use `host.j2` to create specific configuration for host where `mac_address` is defined in the inventory
- Assemble all files generated in the previous actions.

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/ztp-servers/ztp.yaml](../group_vars/ztp-servers)
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
      build:          # Local directory where to store configuration file
```

Inventory file must contains the following information for all hosts that need to be defined in your ZTP server:

***[hosts.ini](../../hosts.ini):***
```
<Your Device Name>       junos_host=<mgt_ip>    mac_address=<device_mac_address>
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
		option tftp-server-name "{{ztp.configuration.server.tftp}}";
		option host-name "{{inventory_hostname}}";
		option log-servers {{ztp.configuration.server.log}};
		option ntp-servers {{ztp.configuration.server.ntp}};
		option NEW_OP.image-file-name "{{ztp.path.soft}}/jinstall-ex-4200-13.2R1.1-domestic-signed.tgz";
		option NEW_OP.transfer-mode "{{ztp.configuration.method}}";
		option NEW_OP.config-file-name "{{ztp.path.ftp_conf}}/{{inventory_hostname}}.config";
	}
```
