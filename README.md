# Webarchitects Ansible Synapse Role

This repo will contain an Ansible role to install, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/), see also the [synapse-server](https://git.coop/webarch/synapse-server) repo for an example of how to use this role.

## Synapse configuration

This role used `debconf` to set the `server_name` and `report_stats` variables, which results in values for these variables being written to files in `/etc/matrix-synapse/conf.d`.

The `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables are randomly generated on the server and also written to files in  `/etc/matrix-synapse/conf.d`.

The `database` dictionary is defined in the [defaults/main.yml](defaults/main.yml) file and is written to `/etc/matrix-synapse/conf.d/database.yaml`.

The main configuration file, `/etc/matrix-synapse/homeserver.yaml` is backed up to `/etc/matrix-synapse/.homeserver.original.bak.yaml` and then replaced with the `synapse_homeserver` dictionary set via the [defaults/main.yml](defaults/main.yml).

## Defaults

| Variable name        | Default value    | Comment                                                                        |
|----------------------|------------------|--------------------------------------------------------------------------------|
| `synapse`            | `false`          | Set to `true` to run the tasks in this role.                                   |
| `synapse_source`     | `debian`         | Source for the synapse `.deb` package, `debian` requires backports on Bullseye |
