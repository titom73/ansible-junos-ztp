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

At that time, this playbook check if devices are running with the version you defined in your inventory. If the version is not the same, it raises an error per host, otherwise, everything should be success:

* Correct version:

```
tom:ansible-junos-ztp/ (devR1.2*) $ ansible-playbook playbook-junos-check.yml                                                                                                                                  [9:35:28]

PLAY [Check status of Junos devices] *******************************************

TASK [setup] *******************************************************************
ok: [vSRX-GW-LAB]

TASK [junos-check : Check netconf connectivity] ********************************
ok: [vSRX-GW-LAB]

TASK [junos-check : Retrieve information from devices running Junos OS] ********
ok: [vSRX-GW-LAB]

TASK [junos-check : version is OK] *********************************************
ok: [vSRX-GW-LAB] => {
    "msg": "vSRX-GW-LAB (79931958d82c) is running 12.1X47-D35.2"
}

TASK [junos-check : version mismatch] ******************************************
skipping: [vSRX-GW-LAB]

PLAY RECAP *********************************************************************
vSRX-GW-LAB                : ok=4    changed=0    unreachable=0    failed=0
```

* Wrong version example:

```
tom:ansible-junos-ztp/ (devR1.2*) $ ansible-playbook playbook-junos-check.yml                                                                                                                   [9:32:05]

PLAY [Check status of Junos devices] *******************************************

TASK [setup] *******************************************************************
ok: [vSRX-GW-LAB]

TASK [junos-check : Check netconf connectivity] ********************************
ok: [vSRX-GW-LAB]

TASK [junos-check : Retrieve information from devices running Junos OS] ********
ok: [vSRX-GW-LAB]

TASK [junos-check : version is OK] *********************************************
skipping: [vSRX-GW-LAB]

TASK [junos-check : version mismatch] ******************************************
fatal: [vSRX-GW-LAB]: FAILED! => {"changed": false, "failed": true, "msg": "vSRX-GW-LAB (79931958d82c) is running 12.1X47-D35.2 (expected 12.3R3.4)"}

NO MORE HOSTS LEFT *************************************************************

PLAY RECAP *********************************************************************
vSRX-GW-LAB                : ok=3    changed=0    unreachable=0    failed=1
```
