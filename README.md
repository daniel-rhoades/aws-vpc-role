[![Circle CI](https://circleci.com/gh/daniel-rhoades/aws-vpc-role.svg?style=svg&circle-token=8197ca6e8362d9b5f1b78f43174d8ba7857c9b7a)](https://circleci.com/gh/daniel-rhoades/aws-vpc-role)

aws-vpc
=======

Ansible role for simplifying the provisioning and decommissioning of a VPC within an AWS account.

For more detailed on information on the creating VPCs with Ansible see the offical documentation for that module: http://docs.ansible.com/ansible/ec2_vpc_module.html. 

Requirements
------------

Requires the latest Ansible EC2 support modules along with Boto.

You will also need to configure your Ansible environment for use with AWS, see http://docs.ansible.com/ansible/guide_aws.html.

Role Variables
--------------

Defaults:

* vpc_resource_tags: Tags to set on the VPC, by default the name of the VPC is used;
* vpc_internet_gateway: If you want the VPC to be able to directly connect to the Internet, defaults to True;
* vpc_state: Default component states, defaults to `present`.  To delete the VPC set this to `absent`

Required variables:

* vpc_name: You must specify the name of the VPC you wish to create, e.g. my-vpc;
* vpc_region: You must specify the region in which you wish to create the VPC, e.g. eu-west-1;
* vpc_cidr_block: You must specify the CIDR block range you wish the VPC to have, e.g. 172.40.0.0/16;
* vpc_subnets: You must specify the subnets you wish to create, see example playbook section below for further information;
* public_subnet_routes: You must specify the public subnets routes you wish to create, see example playbook section below for further information.

Outputs:

* vpc: The AWS VPC object created as a result of running the `ec2_vpc_module` with the supplied variables.

Dependencies
------------

No dependencies on other roles.

Example Playbook
----------------

Before using this role you will need to install the role, the simplist way to do this is: `ansible-galaxy install daniel-rhoades.aws-vpc`. 

The example playbook below ensures a VPC is provisioned in AWS as specified, e.g. if one already matches the role does nothing, otherwise it gets created.

```
- name: My System | Provision all required infrastructure
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    my_vpc_name: "my_example_vpc"
    my_vpc_region: "eu-west-1"
    my_vpc_cidr: "172.40.0.0/16"
    everywhere_cidr: "0.0.0.0/0"

    # Subnets within the VPC
    my_vpc_subnets:
      - cidr: "172.40.10.0/24"
        az: "{{ my_vpc_region }}a"

      - cidr: "172.40.20.0/24"
        az: "{{ my_vpc_region }}b"

    # Allow the subnets to route to the outside world
    my_public_subnet_routes:
      - subnets:
          - "{{ my_vpc_subnets[0].cidr }}"
          - "{{ my_vpc_subnets[1].cidr }}"
        routes:
          - dest: "{{ everywhere_cidr }}"
            gw: igw
  roles:
    # Provision networking
    - {
        role: daniel-rhoades.aws-vpc,
        vpc_name: "{{ my_vpc_name }}",
        vpc_region: "{{ my_vpc_region }}",
        vpc_cidr_block: "{{ my_vpc_cidr }}",
        vpc_subnets: "{{ my_vpc_subnets }}",
        public_subnet_routes: "{{ my_public_subnet_routes }}"
      }
```

To decommission a VPC:

```
- name: My System | Decommission all required infrastructure
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    my_vpc_name: "my_example_vpc"
    my_vpc_region: "eu-west-1"
    my_vpc_cidr: "172.40.0.0/16"
    everywhere_cidr: "0.0.0.0/0"

    # Subnets within the VPC
    my_vpc_subnets:
      - cidr: "172.40.10.0/24"
        az: "{{ my_vpc_region }}a"

      - cidr: "172.40.20.0/24"
        az: "{{ my_vpc_region }}b"

    # Allow the subnets to route to the outside world
    my_public_subnet_routes:
      - subnets:
          - "{{ my_vpc_subnets[0].cidr }}"
          - "{{ my_vpc_subnets[1].cidr }}"
        routes:
          - dest: "{{ everywhere_cidr }}"
            gw: igw
  roles:
    # Decommission networking
    - {
        role: daniel-rhoades.aws-vpc,
        vpc_state: "absent",
        vpc_name: "{{ my_vpc_name }}",
        vpc_region: "{{ my_vpc_region }}",
        vpc_cidr_block: "{{ my_vpc_cidr }}",
        vpc_subnets: "{{ my_vpc_subnets }}",
        public_subnet_routes: "{{ my_public_subnet_routes }}"
      }
```

License
-------

MIT

Author Information
------------------

Daniel Rhoades (https://github.com/daniel-rhoades)
