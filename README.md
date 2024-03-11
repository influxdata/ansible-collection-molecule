# Ansible Collection - influxdata.molecule

[![Molecule Test](https://github.com/influxdata/ansible-collection-molecule/actions/workflows/main.yml/badge.svg)](https://github.com/influxdata/ansible-collection-molecule/actions/workflows/main.yml)

This collection facilitates the creation and use of test environments using the Ansible [Molecule project](https://ansible.readthedocs.io/projects/molecule/).

It provides tooling to create Molecule testing scenarios via the `init` role, and test platforms via the `docker_platform` role, among others.

When utilizing an image with systemd support (systemd packages are installed, etc.), the `docker_platform` role supports the creation of Docker containers with a functional Systemd implementation, which can be used to test Ansible code that makes use of Systemd services or related functionality.

# Using this collection

The following roles are provided:

* [init](roles/init) - Initialize the Molecule testing framework for a project
* [docker_platform](roles/docker_platform) - Create a docker-based test platform for Molecule
* [prepare_controller](roles/prepare_controller) - Prepare a molecule controller to run local code tests

## Host Requirements

The host from which this collection is run (workstation, CI instance, etc.) must meet the following requirements:

* Molecule [is installed](https://ansible.readthedocs.io/projects/molecule/installation/) and is executable by the user.
* Ansible is installed, and `ansible-playbook` is executable by the user.
  * Ansible can be installed via `pip`, or using many system package managers.
* Docker should be installed
* The user running Molecule should be a member of the `docker` group (able to run `docker` commands without using `sudo`)

In general, Molecule can be installed with the following:

```bash
pip install molecule ansible-lint
```

Ansible can also be installed via `pip` if desired:
```bash
pip install ansible-core
```

Docker CE can be installed by following the appropriate [installation instructions](https://docs.docker.com/engine/install/) for your OS.

## Project requirements

Roles within this collection will attempt to discover what type of project they are being utilized in. This is enabled by setting the appropriate `project_type` configuration variable to `auto`. The project type can also be explicitly specified if this is desired.

Supported project types:  
* `role`
* `collection`
* `monolith`

When used with a role or collection, the Galaxy meta information for the role must be configured!

### Standalone roles

Location: `{{ role_dir }}/meta/main.yml`

For standalone roles that are not part of a collection, the minimum required information is:
```yaml
galaxy_info:
  author: you
  role_name: cool_stuff
```

It is however strongly recommended that the entire file be updated with the correct information for your role!

### Collections

Location: `{{ collection_dir }}/galaxy.yml`

The following minimum values should be defined:  
* `namespace`
* `name`
* `version`

It is however also strongly recommended that the entire file be updated with the correct information for your collection! 

### Monoliths

A monolith is a project that contains both roles and playbooks in the same directory structure.

This collection supports the following file structure:  
```
.
├── ansible.cfg
├── collections
│   ├── ansible_collections
│   └── requirements.yml
├── inventory
├── molecule
│   └── default
│       ├── [...]
│       ├── collections.yml
│       ├── converge.yml
│       ├── molecule.yml
│       └── verify.yml
├── playbooks
│   ├── configure_toaster
│   │   ├── main.yml -> rick.yml
│   │   ├── README.md
│   │   ├── rick.yml
│   │   ├── tasks
│   │   └── vars
│   ├── configure_server
│   │   ├── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   └── templates
│   ├── [...]
│   └── install_things
│       ├── install-cheese.yml
│       ├── tasks
│       └── vars
├── README.md
├── roles
│   ├── powerlevel_9000
│   │   ├── defaults
│   │   ├── meta
│   │   ├── README.md
│   │   ├── tasks
│   │   ├── tests
│   │   └── vars
│   ├── disks
│   │   ├── defaults
│   │   ├── handlers
│   │   ├── meta
│   │   ├── README.md
│   │   ├── tasks
│   │   ├── templates
│   │   ├── tests
│   │   └── vars
│   ├── [...]
│   └── something_else
└── scripts
```

When using this collection within a monolithic repository, make sure that any `ansible.cfg` configuration includes the default ansible locations, or that this collection is installed within the repository.

For example, `ansible.cfg` should contain something like:  
```yaml
[defaults]
roles_path = ./roles:/usr/share/ansible/roles:/etc/ansible/roles:~/.ansible/roles
collections_path = ./collections:/usr/share/ansible/collections:~/.ansible/collections
```

or this collection should be installed locally with:  
```bash
ansible-galaxy collection install -p ./collections influxdata.molecule
```

or if your `collections/requirements.yml` includes this collection:
```bash
ansible-galaxy collection install -p ./collections -r ./collections/requirements.yml
```

#### Testing roles within a monolithic project

When configuring molecule testing for individual roles within a monolithic project (creating a `roles/<role_name>/molecule` directory), take care _not_ to name the scenario "default", as there is already a "default" scenario for the monolithic project itself if you have created `molecule/default` as described above! Instead, name your role scenario with a unique name.

```bash
ROLE_NAME=your_role
mkdir -p molecule/role-$ROLE_NAME
wget -P molecule/role-$ROLE_NAME https://raw.githubusercontent.com/syndr/ansible-collection-molecule/main/roles/init/files/init.yml
ansible-playbook molecule/role-$ROLE_NAME/init.yml
```

# Contributing

Pull requests are welcomed!

