Potato Platform
=========

Create an InfluxDB Enterprise cluster utilizing the Potato orchestrator application

Configuration of this role is done primarily via the `platforms` definition in `molecule.yml`.

The following options are required:

- `name`: The name of the platform
- `type: potato`

The following options are optional:

- `potato_address`: The URL to use for the Potato orchestrator (string, do not include the `/v1/potato` path)
- `influxdb_cluster_config`: The configuration to be passed to the Potato orchestrator to create the InfluxDB cluster (dictionary)
  - Configuration options are found in the [Potato API documentation](https://github.com/influxdata/potato/blob/master/swagger.yml).
- `private_key_source_path`: The path to the (existing) private key to use for SSH access to the InfluxDB Enterprise nodes
- `aws_profile`: The AWS profile to use for authentication (string)
- `boot_wait_seconds`: The number of seconds to wait for the instance to boot (integer)
- `private_key_path`: The path to the private key file for the SSH key pair (string)
- `public_key_path`: The path to the public key file for the SSH key pair (string)
- `security_group_name`: The name of the security group to use for the instance (string)
- `security_group_description`: The description of the security group to use for the instance (string)
- `security_group_rules`: A list of security group rules to apply to the instance (list of dicts)
- `security_group_rules_egress`: A list of security group egress rules to apply to the instance (list of dicts)
- `ssh_user`: The SSH user to use for connecting to the instance (string)
- `ssh_port`: The SSH port to use for connecting to the instance (integer)
- `security_groups`: A list of security group names to apply to the instance (list of strings)
- `tags`: A dictionary of tags to apply to the instance (dictionary)
- `vpc_id`: The ID of the VPC to use for connectivity between Molecule and the cluster hosts (string)
- `vpc_subnet_id`: The ID of the subnet to use to find the VPC (string, mutually exclusive with `vpc_id`)

Requirements
------------

The Potato orchestrator application must be installed, configured, and able to create InfluxDB Enterprise clusters.

Collections:
- amazon.aws

The controller will need the following Python libraries:
- boto3

Role Variables
--------------

When using this role, which should be done via the [platform role](../platform), you will need the following variables defined:

- `potato_platform_potato_username`: The username to use for the Potato orchestrator 
  - This can also be set using the `POTATO_API_USERNAME` environment variable, which will take precedence
- `potato_platform_potato_password`: The password to use for the Potato orchestrator
  - This can also be set using the `POTATO_API_PASSWORD` environment variable, which will take precedence

In order to connect to AWS, you will need the following environment variables to be set:

```bash

 export AWS_ACCESS_KEY_ID="blahblahblahblah"
 export AWS_SECRET_ACCESS_KEY="hurpderpherpderpdpeypedpderpyderp"
 export AWS_SESSION_TOKEN="hurpderpherpderpdpeypedpderpyderpahahwhizbanglotsofstuffblablablabla"
```

The `AWS_SESSION_TOKEN` variable is only required if you are using temporary credentials.

Full role configuration options are available in the [defaults/main.yml](defaults/main.yml) file.

Dependencies
------------

None

Example Playbook
----------------

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

```yaml
platforms:
  - name: potato-cluster
    type: potato
    vpc_id: "vpc-12345678"
    potato_address: "http://potato.example.com:8080"
    influxdb_cluster_config:
      region: us-west-2
      username: influxdb-admin
      password: supersecretpassword
      init: latest
      config_tag: 1.11.6.1-staging-pro
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

- InfluxData

