molecule.ec2_platform
=========

Create an Amazon EC2-based test platform for Molecule.

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

Required configuration options are:

- `name`: The name of the platform (string)
- `type`: `ec2`
- `image`: The AMI ID to use for the instance (string)
- `region`: The AWS region to deploy the instance in (string)
- `vpc_id`: The VPC ID to deploy the instance in (string)
- `vpc_subnet_id`: The VPC subnet ID to deploy the instance in (string)

Optional configuration options are:

- `assign_public_ip`: Whether or not to assign a public IP to the instance (boolean)
- `aws_profile`: The AWS profile to use for authentication (string)
- `boot_wait_seconds`: The number of seconds to wait for the instance to boot (integer)
- `instance_type`: The instance type to use for the instance (string)
- `key_inject_method`: The method to use for injecting the SSH key into the instance ("cloud-init"/"ec2")
- `key_name`: The name of the SSH key pair to use for the instance (string)
- `private_key_path`: The path to the private key file for the SSH key pair (string)
- `public_key_path`: The path to the public key file for the SSH key pair (string)
- `security_group_name`: The name of the security group to use for the instance (string)
- `security_group_description`: The description of the security group to use for the instance (string)
- `security_group_rules`: A list of security group rules to apply to the instance (list of dicts)
- `security_group_rules_egress`: A list of security group egress rules to apply to the instance (list of dicts)
- `ssh_user`: The SSH user to use for connecting to the instance (string)
- `ssh_port`: The SSH port to use for connecting to the instance (integer)
- `cloud_config`: The cloud-config data to use for the instance (dictionary)
- `image_name`: The name of the image to use for the instance (string)
- `image_owner`: The owner of the image to use for the instance (string)
- `security_groups`: A list of security group names to apply to the instance (list of strings)
- `tags`: A dictionary of tags to apply to the instance (dictionary)
- `volumes`: A list of volumes to attach to the instance (list of dicts)

Requirements
------------

**Python Modules**
- `boto3`

Role Variables
--------------

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

**Collections**
- `amazon.aws`

Example Playbook
----------------

This role is intended to be used via the `molecule.platform` role that is included with this collection, and should not be referenced directly in a playbook.

Configuration is done via the `platforms` section of the `molecule.yml` file in your Molecule scenario directory.

```yaml
platforms:
  - name: ec2-rockylinux9
    type: ec2
    image: "ami-067daee80a6d36ac0"
    instance_type: "t3.micro"
    region: "us-east-2"
    vpc_id: "vpc-12345678"
    vpc_subnet_id: "subnet-12345678"
```

To utilize this role, use the `platform` role that is included with this collection in your `create.yml` playbook!

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

