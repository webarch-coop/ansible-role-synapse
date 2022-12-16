# Webarchitects Ansible Synapse Role

[![pipeline status](https://git.coop/webarch/synapse/badges/main/pipeline.svg)](https://git.coop/webarch/synapse/-/commits/main)

This repo will contain an Ansible role to install, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/), see also the [synapse-server](https://git.coop/webarch/synapse-server) repo for an example of how to use this role.

## Synapse configuration

### Debian debconf

This role uses `debconf` to set the `server_name` and `report_stats` variables, which results in values for these variables being written to files in `/etc/matrix-synapse/conf.d`.

## Homeserver configuration

The main configuration file, `/etc/matrix-synapse/homeserver.yaml` is backed up to `/etc/matrix-synapse/.homeserver.original.bak.yaml` and then replaced with the `synapse_homeserver` dictionary set via the [defaults/main.yml](defaults/main.yml).

#### Database configuration

See `synapse_db` below.

#### Synapse secret strings

The `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables are randomly generated on the server and written to files in  `/etc/matrix-synapse/conf.d`.

## Defaults

#### synapse

A boolean, set `synapse` to `true` to run the tasks in this role.

#### synapse_db

A dictionary containing the database configuration details, which are written to `/etc/matrix-synapse/conf.d/database.yaml`, these settings can be fully replaced via group or host variables or partially replaced using [YAML anchors and aliases](https://docs.ansible.com/ansible/7/playbook_guide/playbooks_advanced_syntax.html#yaml-anchors-and-aliases-sharing-variable-values), for example:

```yaml
synapse_db:
  <<: *synapse_db_defaults
  database:
    name: "sqlite3"
    args:
      database: "/var/lib/matrix-synapse/homeserver.db"
```

#### synapse_client_domain

The domain name that clients will use to connect to the server on port 443, `synapse_client_domain` defaults to `matrix.{{ inventory_hostname }}`.

#### synapse_federation_domain

The domain name that other server will use to connect to on port 8448, `synapse_federation_domain` defaults to `{{ inventory_hostname }}`.

#### synapse_source

The source of the `matrix-synapse` package, this curently defaults to `debian`, in the future `synapse` might also be supported for installing the upstream package.

#### synapse_homeserver

A dictionary to be written as the main configuration file to `/etc/matrix-synapse/homeserver.yaml`, these settings can be fully replaced via group or host variables or partially replaced using [YAML anchors and aliases](https://docs.ansible.com/ansible/7/playbook_guide/playbooks_advanced_syntax.html#yaml-anchors-and-aliases-sharing-variable-values), for example:

```yaml
synapse_homeserver:
  <<: *synapse_homeserver_defaults
  report_stats: true

```

## Notes

Generate a example config file:

```bash
usr/bin/python3 -m synapse.app.homeserver --config-path=/etc/matrix-synapse/test.yml \
  --generate-config --report-stats=no -H $(hostname -f)
```
