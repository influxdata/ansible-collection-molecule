influxdata.molecule.init
=========

Initialize the Molecule testing framework for a project.

# Usage

This role will primarily be run directly via the `ansible-playbook` command in order to bootstrap a test environment.

The following are instructions on how to do so.

## Create the molecule scenario directory

```bash
mkdir -p molecule/default
```

## Create the init playbook

Within the new scenario directory (default), create a file `init.yml` containing the init playbook located at [files/init.yml](files/init.yml).

Alternatively, you can run:
```bash
wget -P molecule/default https://raw.githubusercontent.com/influxdata/ansible-collection-molecule/main/roles/init/files/init.yml
```

You should now have a directory structure similar to the following:
```
my_cool_role
├── defaults
├── molecule
│   └── default
│       └── init.yml
├── handlers
├── LICENSE
├── meta
├── README.md
├── tasks
├── templates
└── vars
```


## Edit the init playbook

Edit the configuration variables as desired for your deployment.

## Run the init playbook

```bash
ansible-playbook molecule/default/init.yml
```

You should now see that additional configuration has been added to the `default` scenario directory:  
```
my_cool_role
├── defaults
├── molecule
│   ├── resources
│   │   ├── cleanup.yml
│   │   ├── collections.yml
│   │   ├── converge.yml
│   │   ├── create.yml
│   │   ├── destroy.yml
│   │   ├── prepare.yml
│   │   ├── requirements.yml
│   │   ├── side_effect.yml
│   │   └── verify.yml
│   └── role-disks-docker
│       ├── collections.yml
│       ├── init.yml
│       ├── molecule.yml
│       └── requirements.yml
├── handlers
├── LICENSE
[...]
```

## Update configuration

The 'platform' configuration for the Molecule scenario can be updated in the `molecule/default/molecule.yml` file as needed.

The 'converge' playbook can be updated in the `molecule/resources/converge.yml` file as needed.


## Run Molecule to verify initial setup

Your base molecule scenario should now be deployed and ready to use. To verify this, you can run molecule against the new scenario:

```bash
molecule test
```

Note: The `molecule` command must be run from within the extensions directory. Furthermore, if you are creating additional scenarios with names other `default`, you will need to include the scenario name when running Molecule. This can be done as:  
```bash
molecule test -s my_other_scenario
```

Molecule should run through its test process and exit successfully. Success should be apparent from the lack of red text in the output, but can also be verified from the shell by checking the exit code of the `molecule` command:

```bash
❯ echo $?                                         
0
```

Requirements
------------

* Molecule should be installed and executable from a location in the users PATH
* Ansible should be installed, with `ansible-playbook` executable via the users PATH
* Docker should be installed
* The current user should be a member of the `docker` group

Configuration
------------

This role should contain reasonably sane defaults. The most often configured value will most likely be `init_platforms`, which contains a list with each list item containing the configuration for an individual test platform (IE Rocky Linux 9 or Amazon Linux 2023).

For example:
```yaml
init_platforms:
  - name: docker-rocklinux9
    type: docker
    image: "geerlingguy/docker-rockylinux9-ansible:latest"
    systemd: true
```

The name of each platform should be unique, and other Systemd-enabled OS containers can be found [here](https://hub.docker.com/search?q=geerlingguy%2Fdocker-).

Currently supported platform types are:

* [docker](../docker_platform/README.md)

Role Variables
--------------

```yaml
# The type of project that this Molecule configuration will be integrated into (role, collection, playbook, monolith)
init_project_type: auto

# The type of platform that this Molecule configuration will be testing on (docker, ec2)
# WARN: mixing platform types is not supported!
init_platform_type: docker

# Version of this collection that should be used by the Molecule test
# - Set to "current" to attempt to use the running version
init_collection_version: latest

# Source of the collection that this role is part of (galaxy, git)
init_collection_source: git

# Path to the ansible secret file that should be used by the Molecule test
#  - Variable substitution can be used as described here: https://ansible.readthedocs.io/projects/molecule/configuration/#variable-substitution
#  - Set to "" to disable
init_ansible_secret_path: "{{ lookup('env', 'ANSIBLE_VAULT_PASSWORD_FILE') | default('') }}"

# Platforms that this test configuration should run on
#  list of dicts, each required to contain:
#    name: (string)
#    type: (string)
#    <platform-specific configuration>
#
#  for example, in the case of the "docker" type:
#    - name: docker-platform-with-a-descriptive-name
#      type: docker
#      image: (string, container image path)
#      systemd: (true/false)                # enable systemd in the container
#      privileged: (true/false)             # run the container in privileged mode
#      cpus: (int)                          # number of CPUs to allocate to the container
#      memory: (int)K/M/G                   # amount of memory to allocate to the container
#      published_ports: (list)              # list of ports to publish from the container
#      exec_systemd: (true/false)           # customize the container entrypoint to run systemd
#      exec_systemd_build_commands: (list)  # commands to run when building the systemd-enabled container
#      hostvars: (dict)                     # hostvars to be added to the Ansible inventory
#
#   See README.md in the 'roles/docker_platform' directory for more information
#    - Similarly, other platform types will have their own README.md files
#
# If not specified, the role will attempt to use the default platform configuration
init_platforms: []

# Create backups of any files that would be clobbered by running this role
init_file_backup: true

# Overwrite any existing 'resource' playbook files in the 'molecule/resources' directory
init_overwrite_resources: false

# Initialize ARA support in the Molecule configuration -- https://ara.recordsansible.org/
init_ara_support: true
```

Dependencies
------------

**Collections**  
* influxdata.molecule

Example Playbook
----------------

```yaml
---
# Initialize a Molecule scenario for use within a role

- name: Provision file structure
  hosts: localhost
  tasks:
    - name: Launch provisioner
      ansible.builtin.include_role:
        name: influxdata.molecule.init
      vars:
        init_project_type: auto
        init_platforms:
          - name: docker-amazonlinux2023
            type: docker
            image: geerlingguy/docker-amazonlinux2023-ansible:latest
            systemd: true
            privileged: false
```

License
-------

MIT

Author Information
------------------

- InfluxData

