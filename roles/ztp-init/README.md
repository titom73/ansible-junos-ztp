# 1. ztp-init overview

This role is in charge of creating local directories required to store configuration files before sending them to ZTP server

- Delete building diretory defined with `{{build_dir}}` which is `conf/ztp` by default
- Recreate building directory
- Create subdiretory in `{{build_dir}}` named `configlet` to store partial configuration files

## 1.1. Variables needed by this role:
All variables are defined in [group_vars/all/vars.yaml](../../group_vars/all/vars.yaml)
```
---
build_dir: conf/ztp            # Local directory where to store configuration file
```
