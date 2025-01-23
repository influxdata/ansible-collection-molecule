molecule.docker_platform
=========

Create a docker-based test platform for Molecule.

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

Required configuration options are:

- `name`: Name of the platform (string)
- `type`: `docker`

Optional configuration options of note are:

- `image`: Docker image to use for the platform (string)
- `systemd`: Whether the container should be started with SystemD enabled (boolean)
- `privileged`: Whether the container should be started in privileged mode (boolean)
- `cpus`: Number of CPUs to allocate to the container (integer)
- `memory`: Amount of memory to allocate to the container (string)
- `pull`: Pull policy to use for the image (string)
- `volumes`: List of volumes to mount in the container (list)

Requirements
------------

1. Docker should be installed
1. The current user should be a member of the `docker` group

Role Variables
--------------

Configuration that can be set in the `platforms` section of the `molecule.yml` file in your Molecule scenario directory:

```yaml
# Name of the platform in the Molecule configuration
name: Your platform name

# Image to use for the container
image: "geerlingguy/docker-rockylinux9-ansible:latest"

# Run the container in privileged mode (WARNING: Less secure!)
privileged: false

# List of capabilities to add to the container
capabilities: []

# List of capabilities to drop from the container
cap_drop: []

# Command to run in the container
command: ""

# Container hostname
hostname: molecule-ci-{{ __docker_platform_instance.name | default('instance') }}

# Number of CPUs to allocate to the container
cpus: 2

# List of device path and read rate (bytes per second) from device.
#  Each entry should be a dictionary with keys 'path' and 'rate'.
device_read_bps: []

# List of device path and write rate (bytes per second) to device.
#  Each entry should be a dictionary with keys 'path' and 'rate'.
device_write_bps: []

# List of device path and read rate (IO per second) from device.
#  Each entry should be a dictionary with keys 'path' and 'rate'.
device_read_iops: []

# List of device path and write rate (IO per second) to device.
#  Each entry should be a dictionary with keys 'path' and 'rate'.
device_write_iops: []

# List of host device bindings to add to the container.
#  Each binding is a mapping expressed in the format:
#    <path_on_host>:<path_in_container>:<cgroup_permissions>
devices: []

# List of DNS options
dns_opts: []

# List of DNS search domains
dns_search_domains: []

# List of DNS servers
dns_servers: []

# The URL or Unix socket path used to connect to the Docker API
docker_host: "{{ lookup('env', 'DOCKER_HOST') | default('unix:///var/run/docker.sock') }}"

# Dict of host-to-IP mappings, where each host name is a key in the dictionary. Each host name will be added to the container’s /etc/hosts file.
#  Instead of an IP address, the special value host-gateway can also be used, which resolves to the host’s gateway
#  IP and allows containers to connect to services running on the host.
etc_hosts: {}

# Dict of labels to apply to the container
labels: {}

# Memory limit in format <number>[<unit>].
#  Number is a positive integer.
#  Unit can be B (byte), K (kibibyte, 1024B), M (mebibyte), G (gibibyte), T (tebibyte), or P (pebibyte).
memory: "0"

# Connect the container to a network.
#  Choices are bridge, host, none, container:<name|id>, <network_name> or default.
network_mode: default

# List of ports to publish from the container to the host.
#  Use docker CLI syntax: 8000, 9000:8000, or 0.0.0.0:9000:8000,
#  where 8000 is a container port, 9000 is a host port, and 0.0.0.0 is a host interface.
published_ports: []

# Pull setting for the container image (never,missing,always)
pull: always

# List of security options in the form of "label:user:User"
security_opts: []

# State of the container
state: started

# Whether the container runs systemd
systemd: false

# Attempt to execute systemd in the container on start
#  WARNING:
#  - This can cause issues with some containers
#  - Not required if the container is already built with systemd running as PID 1
#  - Expects the container to have systemd packages present
#  - Rebuilds the container with a custom entrypoint, provided by 'exec_systemd_path'
exec_systemd: false

# Path to the systemd binary in the container
#  - This is only used if 'exec_systemd' is true
exec_systemd_path: /usr/lib/systemd/systemd

# List of commands to run as part of the docker build process to enable systemd
#  - This is only used if 'exec_systemd' is true
#  - Each command should be a string
#  - Commands are run in the order they are defined, using the docker RUN directive
exec_systemd_build_commands: []

# Tmpfs mounts to add to the container
tmpfs: []

# Volumes to mount in the container
#  Use docker CLI-style syntax: /host:/container[:mode]
#
#  Mount modes can be a comma-separated list of various modes such as
#   ro, rw, consistent, delegated, cached, rprivate, private, rshared, shared, rslave, slave, and nocopy.
#
#   Note that the docker daemon might not support all modes and combinations of such modes.
#
# SELinux hosts can additionally use z or Z to use a shared or private label for the volume.
volumes: []
```

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

# we want to avoid errors like "Failed to create temporary directory"
- name: Validate molecule inventory
  hosts: molecule
  gather_facts: false
  tasks:
    - name: Check kernel version
      ansible.builtin.raw: uname -a
      register: result
      changed_when: false

    - name: Display kernel info
      ansible.builtin.debug:
        msg: "{{ result.stdout }}"
```


License
-------

MIT

Author Information
------------------

- InfluxData

