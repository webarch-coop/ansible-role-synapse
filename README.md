# Webarchitects Ansible Synapse Role

[![pipeline status](https://git.coop/webarch/synapse/badges/main/pipeline.svg)](https://git.coop/webarch/synapse/-/commits/main)

This repo contains an Ansible role to install, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/), see also the [synapse-server](https://git.coop/webarch/synapse-server) repo for an example of how to use this role.

## Synapse configuration

This role uses `debconf` to set the `server_name` and `report_stats` variables, which results in values for these variables being written to files in `/etc/matrix-synapse/conf.d`.

The `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables are randomly generated on the server and written to files in  `/etc/matrix-synapse/conf.d`.

## Defaults

#### synapse

A boolean, set `synapse` to `true` to run the tasks in this role.

#### synapse_db

A dictionary containing the database configuration details, which are written to `/etc/matrix-synapse/conf.d/database.yaml`, for example to use SQLite  rather than PostgreSQL:

```yaml
synapse_db:
  database:
    name: "sqlite3"
    args:
      database: "/var/lib/matrix-synapse/homeserver.db"
```

#### synapse_homeserver

A required dictionary to be written as the main configuration file to `/etc/matrix-synapse/homeserver.yaml` see [the documentation](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html) for all the options.

The dictionary for `synapse_homeserver` in the [defaults/main.yml](defaults/main.yml) contains comments that are not written to the server.

#### synapse_homeserver_combine

A optional dictionary that is combined with the `synapse_homeserver` dictionary using the [ansible.builtin.combine filter](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/combine_filter.html) with the `list_merge` parameter set to `replace` and `recursive` set to `true`, this means that individual variables in the `synapse_homeserver` dictionary can be replaced without having to redefine all the defaults.

By default `synapse_homeserver_combine` is not defined, an example of how it can be used, to set the `web_client_location` variable in `/etc/matrix-synapse/homeserver.yaml`:

```yaml
synapse_homeserver_combine:
  web_client_location: "https://{{ element_domain }}/"
```

The result of combining `synapse_homeserver` and `synapse_homeserver_combine` is defined as `synapse_homeserver_proposed_config`, the existing `/etc/matrix-synapse/homeserver.yaml` confguration is read and defined as `synapse_homeserver_existing_config`.

#### synapse_server_name

The domain name to use for the [server_name](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html#server_name), this is used for addresses, for example if the `server_name` was `example.com`, usernames on your server would be in the format `@user:example.com`.

#### synapse_source

The source of the `matrix-synapse` package, this curently defaults to `debian`, in the future `synapse` might also be supported for installing the upstream package.

## Single sign-on

Note that if `synapse_homeserver_combine` is defined then variables set in `synapse_homeserver` can't use Jinja2 escaping, for example:

```yaml
        localpart_template: "{% raw %}{{ user.preferred_username }}{% endraw %}"
```

As it will result in Ansible looking for the `user` variable, for this reason the default SSO settings are written to `/etc/matrix-synapse/conf.d/sso.yaml.example`, move this file to `/etc/matrix-synapse/conf.d/sso.yaml` and edit it as required to configure SSO.

## Notes

Generate a example config file:

```bash
usr/bin/python3 -m synapse.app.homeserver --config-path=/etc/matrix-synapse/test.yml \
  --generate-config --report-stats=no -H $(hostname -f)
```
