# Ansible Collection - influxdata.molecule

[![Molecule Test](https://github.com/influxdata/ansible-collection-molecule/actions/workflows/main.yml/badge.svg)](https://github.com/influxdata/ansible-collection-molecule/actions/workflows/main.yml)

This collection facilitates the creation and use of test environments using the Ansible [Molecule project](https://ansible.readthedocs.io/projects/molecule/).

It provides tooling to create Molecule testing scenarios via the `init` role, and test platforms via the `docker_platform` role, among others.

When utilizing an image with systemd support (systemd packages are installed, etc.), the `docker_platform` role supports the creation of Docker containers with a functional Systemd implementation, which can be used to test Ansible code that makes use of Systemd services or related functionality.

# What is Molecule?

> Molecule project is designed to aid in the development and testing of Ansible roles.

Molecule is a testing platform for your Ansible projects that enables testing of your code both during development and after release via CI infrastructure.

Some resources on Molecule can be found here:
* [Developing and Testing Ansible Roles with Molecule and Podman - Part 1](https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1)
* [Testing your Ansible roles with Molecule](https://www.jeffgeerling.com/blog/2018/testing-your-ansible-roles-molecule)
* [Ansible Collections: Role Tests with Molecule](https://ericsysmin.com/2020/04/30/ansible-collections-role-tests-with-molecule/)
* [Introducing Ansible Molecule with Ansible Automation Platform](https://developers.redhat.com/articles/2023/09/13/introducing-ansible-molecule-ansible-automation-platform#an_automation_testing_framework_built_for_the_enterprise)

> [!WARNING]  
> Some [fairly significant changes](https://ansible.readthedocs.io/projects/molecule/next/) have been made in Molecule v6. Most noticable among these are likely to be that `ansible` is now the only driver included by default (previously called `delegated`), and that the `molecule init` command now only supports creation of scenarios, not Ansible roles. 
>
> This [RedHat article](https://developers.redhat.com/articles/2023/09/13/introducing-ansible-molecule-ansible-automation-platform#) has some more information on this change.
>
> When reading the above referenced articles, keep in mind their publishing dates, and that there may have been breaking changes to Molecule's functionality since that time!

# Using this collection

The following roles are provided:

* [init](roles/init) - Initialize the Molecule testing framework for a project
* [docker_platform](roles/docker_platform) - Create a docker-based test platform for Molecule
* [prepare_controller](roles/prepare_controller) - Prepare a molecule controller to run local code tests

The recommended way to use this collection is to provision Molecule scenarios using the [init role](roles/init). The `init` role provides template configurations that will work in various project types.

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

The `init` role from this collection will attempt to discover what type of project it is being utilized in. This is enabled by setting the `init_project_type` configuration variable to `auto`. The project type can also be explicitly specified if this is desired.

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

#### Testing roles and playbooks within a monolithic project

When configuring molecule testing for individual roles or playbooks within a monolithic project (creating a `roles/<role_name>/molecule` or `playbooks/<playbook_name>/molecule` directory), take care _not_ to name the scenario "default", as there is already a "default" scenario for the monolithic project itself if you have created `molecule/default` as described above! Instead, name your role scenario with a unique name.

For example (role):

```bash
ROLE_NAME=your_role
mkdir -p molecule/role-$ROLE_NAME
wget -P molecule/role-$ROLE_NAME https://raw.githubusercontent.com/influxdata/ansible-collection-molecule/main/roles/init/files/init.yml
ansible-playbook molecule/role-$ROLE_NAME/init.yml
```

Note that in this circumstance, you will need to specify the scenario name in order to run molecule against it (as it is not named `default`).

> [!WARNING]  
> Creating more than one `default` scenario within a repository (IE: within individual roles) will cause Molecule to fail to run at the outer project level!

Running the `molecule list` command will provide you an overview of the available scenarios

```bash
❯ molecule list                     
INFO     Running pb-example_playbook > list
                     ╷             ╷                  ╷                     ╷         ╷            
  Instance Name      │ Driver Name │ Provisioner Name │ Scenario Name       │ Created │ Converged  
╶────────────────────┼─────────────┼──────────────────┼─────────────────────┼─────────┼───────────╴
  docker-rockylinux9 │ default     │ ansible          │ pb-example_playbook │ false   │ false      
                     ╵             ╵                  ╵                     ╵         ╵
```

And running the full test suite for this playbook would be done as:

```bash
molecule test -s pb-example_playbook
```

While running just the "converge" steps (IE: during development) would be:

```bash
molecule converge -s pb-example_playbook
```

> [!TIP]  
> The `molecule list` command will show multiple scenarios when run in the root of a monolithic project that also has molecule configured on individual playbooks or roles contained within it. Note that you will, however, still need to be in the appropriate role or playbook directory in order to successfully run these scenarios!

# Contributing

Pull requests are welcomed!

