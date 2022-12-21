# Webarchitects Ansible Synapse Role

[![pipeline status](https://git.coop/webarch/synapse/badges/main/pipeline.svg)](https://git.coop/webarch/synapse/-/commits/main)

This repo contains an Ansible role to install, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/), see also the [synapse-server](https://git.coop/webarch/synapse-server) repo for an example of how to use this role.

## Dependencies

This role includes the following Webarchitects roles:

* [acmesh](https://git.coop/webarch/acmesh)
* [postgresql](https://git.coop/webarch/postgresql)

## Synapse configuration

This role uses `debconf` to set the `server_name` and `report_stats` variables, which results in values for these variables being written to files in `/etc/matrix-synapse/conf.d`.

In addition the `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables are randomly generated by this role, on the server, and then written to files in  `/etc/matrix-synapse/conf.d`.

The database configuration is also written to a file in `/etc/matrix-synapse/conf.d`, see `synapse_db` below.

The main `/etc/matrix-synapse/homeserver.yaml` configuration file variables can either be set using `synapse_homeserver` to define everything (apart from the variables noted above) or `synapse_homeserver_combine` can be set to overwrite parts of the `synapse_homeserver` dictionary.

Note that since the SSO configuration section of `/etc/matrix-synapse/homeserver.yaml` uses Jinga2 templating this configuration is probably best done manually, more on this below.

## Role variables

The following [defaults/main.yml](defaults/main.yml) default variables are set by this role.

#### synapse

`synapse` is a required boolean, set it to `true` to run the tasks in this role, this variable defaults to `false`.

#### synapse_db

`synapse_db` is a dictionary containing the database configuration details, which are written to `/etc/matrix-synapse/conf.d/database.yaml`, for example to use SQLite  rather than PostgreSQL:

```yaml
synapse_db:
  database:
    name: "sqlite3"
    args:
      database: "/var/lib/matrix-synapse/homeserver.db"
```

#### synapse_homeserver

`synapse_homeserver` is a required dictionary that is written, as the main `matrix-synapse` configuration file to `/etc/matrix-synapse/homeserver.yaml` see [the documentation](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html) for all the options.

The original `/etc/matrix-synapse/homeserver.yaml` file that is created by the `matrix-synapse` `.deb` package is moved to `/etc/matrix-synapse/.homeserver.original.bak.yaml` before this role overwrites the configuration.

The dictionary for `synapse_homeserver` in [defaults/main.yml](defaults/main.yml) contains comments which are omitted when `/etc/matrix-synapse/homeserver.yaml` is written to the server.

You can either define a full set of configuration using `synapse_homeserver` (apart from the `synapse_db`, `form_secret`, `macaroon_secret_key` and `registration_shared_secret` variables) or use the `synapse_homeserver_combine` dictionary to overwrite specific configuration in the `synapse_homeserver` dictionary.

#### synapse_homeserver_combine

`synapse_homeserver_combine` is an optional dictionary that is combined with the `synapse_homeserver` dictionary using the [ansible.builtin.combine filter](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/combine_filter.html) with the `list_merge` parameter set to `replace` and `recursive` set to `true`, this means that individual variables in the `synapse_homeserver` dictionary can be replaced without having to redefine all the defaults.

By default `synapse_homeserver_combine` is not defined, an example of how it can be used, to set the `web_client_location` variable in `/etc/matrix-synapse/homeserver.yaml`:

```yaml
synapse_homeserver_combine:
  web_client_location: "https://{{ element_domain }}/"
```

Internally to this role, the result of combining `synapse_homeserver` and `synapse_homeserver_combine` is defined as `synapse_homeserver_proposed_config`, the existing `/etc/matrix-synapse/homeserver.yaml` confguration is read and defined as `synapse_homeserver_existing_config`.

#### synapse_server_name

`synapse_server_name` is a required domain name that is used for the [server_name](https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html#server_name) and is set using [debconf](https://wiki.debian.org/debconf), is written to `/etc/matrix-synapse/conf.d/server_name.yaml` and is used for addresses, for example if the `server_name` was `example.com`, usernames on your server would be in the format `@user:example.com`.

#### synapse_source

`synapse_source` is a required variable for the source of the `matrix-synapse` package, this curently defaults to `debian`, in the future `synapse` might also be supported for installing the upstream package.

## Single sign-on

Note that if `synapse_homeserver_combine` is defined then variables set in `synapse_homeserver` can't use Jinja2 escaping, for example:

```yaml
        localpart_template: "{% raw %}{{ user.preferred_username }}{% endraw %}"
```

As it will result in Ansible looking for the `user` variable, for this reason the default SSO settings are written to `/etc/matrix-synapse/sso_example.yaml`, move this file to `/etc/matrix-synapse/conf.d/sso.yaml` and edit it as required to configure SSO.

## Notes

Generate an example config file:

```bash
usr/bin/python3 -m synapse.app.homeserver --config-path=/etc/matrix-synapse/test.yml \
  --generate-config --report-stats=no -H $(hostname -f)
```

## Dependencies

This role requires Ansible `2.13` or newer plus [JC](https://pypi.org/project/jc/) and [JMESPath](https://pypi.org/project/jmespath/) to be installed using `pip3` on the Ansible controller.

## Repository

The primary URL of this repo is [`https://git.coop/webarch/synapse`](https://git.coop/webarch/synapse) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-synapse) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/synapse).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/synapse/-/releases).

## License

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
