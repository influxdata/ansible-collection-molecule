molecule.platform
=========

Deploy a Molecule platform for testing.

This role handles both the creation and destruction of a Molecule platform, as well as configuration of internal Molecule inventory files necessary to utilize them. It is the recommended way to utilize this collection's platform roles.

Supported platforms are:
- `docker`
- `ec2`

Requirements
------------

1. Molecule should be installed and executable from a location in the users PATH
1. Ansible should be installed, with `ansible-playbook` executable via the users PATH

Role Variables
--------------

```yaml
# Name of this Molecule platform
platform_name: instance

# Whether this platform should be deployed on the current system (present/absent)
platform_state: present

# What type of platform should be deployed
platform_type: docker

# Molecule platform configuration
platform_molecule_cfg: {}
```

Dependencies
------------

**Roles included with this collection:**
- `molecule.docker_platform`
- `molecule.ec2_platform`

Example Playbook
----------------

```yaml
- name: Create
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Create platform(s)
      ansible.builtin.include_role:
        name: syndr.molecule.platform
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

