# 1. ztp-init overview

This role is in charge of creating local directories required to store configuration files before sending them to ZTP server

- Delete building diretory defined with `{{ztp.path.build}}` which is `config/ztp` by default
- Recreate building directory
- Create subdiretory in `{{ztp.path.build}}` named `configlet` to store partial configuration files

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/ztp-servers/ztp.yaml](../../group_vars/ztp-servers)
```
---
ztp:
  path:               # All paths required by ZTP roles
    build:            # Local directory where to store configuration file
```
