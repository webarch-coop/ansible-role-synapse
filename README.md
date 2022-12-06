# Webarchitects Ansible Synapse Role

This repo will contain an Ansible role to install, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/), see also the [synapse-server](https://git.coop/webarch/synapse-server) repo for an example of how to use this role.

## Synapse configuration

### Debian debconf

This role uses `debconf` to set the `server_name` and `report_stats` variables, which results in values for these variables being written to files in `/etc/matrix-synapse/conf.d`.

### Synapse secret strings

The `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables are randomly generated on the server and written to files in  `/etc/matrix-synapse/conf.d`.

## Database configuration

The `database` dictionary is defined in the [defaults/main.yml](defaults/main.yml) file and is written to `/etc/matrix-synapse/conf.d/database.yaml`, these settings can be fully replaced via group or host variables or partially replaced using [YAML anchors and aliases](https://docs.ansible.com/ansible/7/playbook_guide/playbooks_advanced_syntax.html#yaml-anchors-and-aliases-sharing-variable-values), for example:

```yaml
synapse_db:
  <<: *synapse_db_defaults
  database:
    name: "sqlite3"
    args:
      database: "/var/lib/matrix-synapse/homeserver.db"
```

## Homeserver configuration

The main configuration file, `/etc/matrix-synapse/homeserver.yaml` is backed up to `/etc/matrix-synapse/.homeserver.original.bak.yaml` and then replaced with the `synapse_homeserver` dictionary set via the [defaults/main.yml](defaults/main.yml).

## Defaults

| Variable name        | Default value    | Comment                                                                        |
|----------------------|------------------|--------------------------------------------------------------------------------|
| `synapse`            | `false`          | Set to `true` to run the tasks in this role.                                   |
| `synapse_source`     | `debian`         | Source for the synapse `.deb` package, `debian` requires backports on Bullseye |
