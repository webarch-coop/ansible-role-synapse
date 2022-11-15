# Webarchitects Ansible Synapse Role

This repo will contain an Ansible role to instal, configure and upgrade [synapse](https://matrix-org.github.io/synapse/latest/).

## Defaults

| Variable name        | Default value    | Comment                                                                        |
|----------------------|------------------|--------------------------------------------------------------------------------|
| `synapse`            | `false`          | Set to `true` to run the tasks in this role.                                   |
| `synapse_source`     | `debian`         | Source for the synapse `.deb` package, `debian` requires backports on Bullseye |
