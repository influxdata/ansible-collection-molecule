molecule.docker_platform
=========

Create a docker-based test platform for Molecule.

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

Required configuration options are:

- `name`: Name of the platform (string)
- `type`: `docker`
- `image`: Docker image to use for the platform (string)

Optional configuration options are:

- `systemd`: Whether the container should be started with SystemD enabled (boolean)
- `modify_image`: Whether the provided image should be modified at runtime (boolean)
- `modify_image_buildpath`: Path to Docker build files that should be used to modify the image (string)

Requirements
------------

1. Docker should be installed
1. The current user should be a member of the `docker` group

Role Variables
--------------

This role should not be used directly in a playbook, and should instead be used via the `molecule.platform` role.

Detailed information on configuration variables for this role can be found in [defaults/main.yml](defaults/main.yml).

Dependencies
------------

**Collections**  
- community.docker

Example Playbook
----------------

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

```yaml
platforms:
  - name: docker-rockylinux9
    type: docker
    image: geerlingguy/docker-rockylinux9-ansible:latest
    systemd: True
    modify_image: False
    privileged: False
    hostvars: {}
```

To utilize this role, use the `platform` role that is included with this collection in your `create.yml` playbook!

```yaml
- name: Create
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Create platform(s)
      ansible.builtin.include_role:
        name: influxdata.molecule.platform
      vars:
        platform_name: "{{ item.name }}"
        platform_state: present
        platform_type: "{{ item.type }}"
        platform_molecule_cfg: "{{ item }}"
      loop: "{{ molecule_yml.platforms }}"
      loop_control:
        label: item.name

# We want to avoid errors like "Failed to create temporary directory"
- name: Validate that inventory was refreshed
  hosts: molecule
  gather_facts: false
  tasks:
    - name: Check uname
      ansible.builtin.raw: uname -a
      register: result
      changed_when: false

    - name: Display uname info
      ansible.builtin.debug:
        msg: "{{ result.stdout }}"
```


License
-------

MIT

Author Information
------------------

- [@syndr](https://github.com/syndr/)

