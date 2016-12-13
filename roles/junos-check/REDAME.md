# 1. junos-check overview

This role is in charge of monitoring results of ZTP process. It connects to all devices listed under `devices` in your inventory file and compares version running on devices against version defined with `junos_version` in [`hosts.ini`](../../hosts.ini)

## 1.1. Variables needed by this role:
All variables are defined in the inventory file: [hosts.ini](../../hosts.ini)

* Define your group of Junos devices under parent group named `devices`

```
[devices:children]
ex
vsrx
```
* For each group, define connection information and junos version as well:

```
[ex:vars]
junos_version= "12.3R3.4"
junos_package= "jinstall-ex-2200-12.3R3.4-domestic.tgz"
ansible_ssh_user=ansible
ansible_ssh_pass='Password123'
```

> Keep in mind you have to provide a `netconf` access in your configuration template with role `junos-create-config`. Indeed, ansible use `netconf` to connect to devices and retrieve information from device. As it is not enable by default, you have to deal with this statement in your template.

An example of how to activate `netconf` access on a Junos device:

```
system {
    login {
      user ansible {
          uid 2008;
          class super-user;
          authentication {
              encrypted-password "..."; ## SECRET-DATA
          }
      }
    }
    services {
        netconf {
            ssh;
        }
    }
}
```
