# NFVbench Ansible role

This repo hosts the `ansible-role-nfvbench` Ansible role.

This role aims to deploy [NFVbench](https://github.com/opnfv/nfvbench) tool inside an SR-IOV VM on a Openstack platform.
The role includes tasks using the `openstack.cloud` Ansible Collection.


## Contents

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->

- [NFVbench Ansible role](#nfvbench-ansible-role)
    - [Contents](#contents)
    - [Installation](#installation)
        - [Install Python and Git](#install-python-and-git)
        - [Install OpenStack SDK](#install-openstack-sdk)
        - [Install ansible-core](#install-ansible-core)
        - [Install Ansible collections](#install-ansible-collections)
        - [Install ansible-role-nfvbench from GitHub](#install-ansible-role-nfvbench-from-github)
    - [Usage](#usage)
        - [Prerequisites](#prerequisites)
        - [Playbooks](#playbooks)
    - [Examples of nfvbench ansible role configuration](#examples-of-nfvbench-ansible-role-configuration)
        - [NFVbench network design - option 1: Physical Function, 3 physnets](#nfvbench-network-design---option-1-physical-function-3-physnets)
        - [NFVbench network design - option 2: Physical Function, 1 physnet](#nfvbench-network-design---option-2-physical-function-1-physnet)
        - [NFVbench network design - option 3: Virtual Function, 3 physnets](#nfvbench-network-design---option-3-virtual-function-3-physnets)
        - [NFVbench network design - option 4: Virtual Function, 3 physnet](#nfvbench-network-design---option-4-virtual-function-3-physnet)
    - [License](#license)

<!-- markdown-toc end -->


## Installation

Several dependencies have to be installed before `ansible-role-nfvbench` can be
used:

  * Python >= 3.6 and its package manager
  * the [Git](http://git-scm.com/) distributed version control system
  * the OpenStack SDK Python package and its Python dependencies including
    [cryptography](https://pypi.org/project/cryptography/) and
    [netifaces](https://pypi.org/project/netifaces/)
  * the `ansible-core` package
  * the `openstack.cloud` Ansible collection
  * the `community.general` Ansible collection

Detailed installation instructions are given below.  We will assume we install a
given version of `ansible-role-nfvbench` and that it is specified in the
`ARN_VERSION` environment variable.  For instance: `ARN_VERSION=0.2.3`

### Install Python and Git

Use your package manager to install Python >= 3.6, Python package manager and
Git.  Example on Alpine Linux:

```bash
apk add python3 py3-pip py3-wheel
apk add git
```

### Install OpenStack SDK

With the distribution package manager, install `openstacksdk` Python
dependencies to avoid the installation of heavy build dependencies.  Example on
Alpine Linux:

```bash
apk add py3-cryptography py3-netifaces
```

Then install `openstacksdk`:

```bash
python3 -m pip install openstacksdk==0.50.0
```

Remarks:

  * OpenStack SDK has to be available to the Python interpreter on both the
    Ansible controller and the Ansible target hosts.  But in the general case,
    the controller and target are the same host since `ansible-role-nfvbench`
    typically targets localhost.

  * The last version of OpenStack SDK is supposed to support all operations on
    all version of OpenStack.  But because OpenStack SDK did not reach a stable
    version number, we cannot blindly install the latest version without risking
    regressions in `ansible-role-nfvbench`.
  
  * Version 0.50.0 of OpenStack SDK is currently used to test
    `ansible-role-nfvbench`, so this is the recommended version at the moment.
    It supports all OpenStack versions up to Victoria (Victoria was initially
    released in October 2020).

### Install ansible-core

There are several ways to install `ansible-core`: with the distribution package
manager, with pip, ...  Example with Alpine Linux package manager:

```bash
apk add ansible-core
```

Remark: `ansible-role-nfvbench` needs few of the roles and collections provided
by the heavy `ansible` package, so it is better to use `ansible-core` and then
install the missing required dependencies.  This makes a huge size difference
when building container images.

### Install Ansible collections

Install the `openstack.cloud` and `community.general` Ansible collections using
the
[requirements.yml](https://github.com/Orange-OpenSource/ansible-role-nfvbench/blob/master/requirements.yml)
file coming with `ansible-role-nfvbench`:

```bash
git clone -b ${ARN_VERSION} https://github.com/Orange-OpenSource/ansible-role-nfvbench.git
ansible-galaxy collection install -r ansible-role-nfvbench/requirements.yml
```

### Install ansible-role-nfvbench from GitHub

Finally install `ansible-role-nfvbench` with:

```bash
ansible-galaxy role install \
    git+https://github.com/Orange-OpenSource/ansible-role-nfvbench.git,${ARN_VERSION}
```

## Usage

### Prerequisites

- This role uses a `clouds.yaml` file for authentication. This file will permit to detect if user has admin rights to Openstack API.
Example of `clouds.yaml`:

```yaml
clouds:
  nfvbench:
    auth:
      auth_url: https://127.0.0.1/v3
      username: nfvbench
      password: Nfvbench-P@ssw0rd1
      project_id: 1aa22b333cc44dd55eeeee6666fffff
      project_name: nfvbench
      project_domain_name: default
      user_domain_name: Default
    cacert: ca.pem
    region_name: "RegionOne"
    interface: "public"
    identity_api_version: 3
```

`Note: Please add this file in the same path as the Ansible playbook file` 

- A certificate can be used for accessing to Openstack API. Please specify `cacert` property in `clouds.yaml` file.

`Note: Please add this certificate file in the same path as the Ansible playbook file` 

- Openstack images are required for NFVbench deployment. These images can be uploaded to Openstack using this role. In this case, replace `image_path` property value in the `default/main.yml` with the accurate path.


### Playbooks

To use this role, please reference the full name of Ansible role, collection name, and module name that you want to use:

```yaml
---
- hosts: localhost
  roles:
    - role: ansible-role-nfvbench
```

## Examples of nfvbench ansible role configuration

Default configuration is available in `defaults/main.yml` file in nfvbench ansible role sources.

This configuration can be overridden by replacing this file or using Ansible `extra_vars`.

`Note: Please find network design options details here :` [NFVbench network design options](docs/nfvbenchvm-network-design.md)

### NFVbench network design - option 1: Physical Function, 3 physnets

See below the parameters modified for this network design:

```
---
# ... see default file to have other parameters

management_networks:
  - network_name: nfvbexternal_vn1
    external: no
    shared: yes
    cleanup: true

    subnet_name: subnet_nfvbenchexternal_vn1
    gateway: 192.168.90.1
    cidr: 192.168.90.0/24
    enable_dhcp: no
    no_gateway_ip: no

    port_name: nfvbench_mgmt_port
    vnic_type: normal
    security_groups: []
    port_security: no

generator_networks:
  - network_name: net_nfvbench_vn1
    physical_network: data2
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true

    topology: loopback_e2e
    
    subnet_name: subnet_nfvbench_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_pf1
    vnic_type: direct-physical
    security_groups: []
    port_security: no

  - network_name: net_nfvbench_vn2
    physical_network: data3
    network_type: vlan
    segmentation_id: 2519
    external: yes
    shared: yes
    cleanup: true

    topology: loopback_e2e
    
    subnet_name: subnet_nfvbench_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_pf2
    vnic_type: direct-physical
    security_groups: []
    port_security: no

loop_vm_networks:
  - network_name: net_nfvbench_loop_vn1
    physical_network: data1
    network_type: vlan
    segmentation_id:
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_loop_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

  - network_name: net_nfvbench_loop_vn2
    physical_network: data1
    network_type: vlan
    segmentation_id:
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_loop_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

```

### NFVbench network design - option 2: Physical Function, 1 physnet

See below the parameters modified for this network design:

```
---
# ... see default file to have other parameters

management_networks:
  - network_name: nfvbexternal_vn1
    external: no
    shared: yes
    cleanup: true

    subnet_name: subnet_nfvbenchexternal_vn1
    gateway: 192.168.90.1
    cidr: 192.168.90.0/24
    enable_dhcp: no
    no_gateway_ip: no

    port_name: nfvbench_mgmt_port
    vnic_type: normal
    security_groups: []
    port_security: no

generator_networks:
  - network_name: net_nfvbench_vn1
    physical_network: data1
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true

    topology: loopback_e2e
    
    subnet_name: subnet_nfvbench_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_pf1
    vnic_type: direct-physical
    security_groups: []
    port_security: no


  - network_name: net_nfvbench_vn2
    physical_network: data1
    network_type: vlan
    segmentation_id: 2519
    external: yes
    shared: yes
    cleanup: true

    topology: loopback_e2e

    subnet_name: subnet_nfvbench_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_pf2
    vnic_type: direct-physical
    security_groups: []
    port_security: no

loop_vm_networks: {{ generator_networks }}
```

### NFVbench network design - option 3: Virtual Function, 3 physnets

See below the parameters modified for this network design:

```
---
# ... see default file to have other parameters

management_networks:
  - network_name: nfvbexternal_vn1
    external: no
    shared: yes
    cleanup: true

    subnet_name: subnet_nfvbenchexternal_vn1
    gateway: 192.168.90.1
    cidr: 192.168.90.0/24
    enable_dhcp: no
    no_gateway_ip: no

    port_name: nfvbench_mgmt_port
    vnic_type: normal
    security_groups: []
    port_security: no

generator_networks:
  - network_name: net_nfvbench_vn1
    physical_network: data2
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true

    topology: loopback_e2e
    
    subnet_name: subnet_nfvbench_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf1
    vnic_type: direct
    security_groups: []
    port_security: no


  - network_name: net_nfvbench_vn1bis
    physical_network: data3
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true

    topology: loopback
    
    subnet_name: subnet_nfvbench_vn1bis
    gateway:
    cidr: 192.168.92.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf1bis
    vnic_type: direct
    security_groups: []
    port_security: no

  - network_name: net_nfvbench_vn2
    physical_network: data3
    network_type: vlan
    segmentation_id: 2519
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf2
    vnic_type: direct
    security_groups: []
    port_security: no

loop_vm_networks:
  - network_name: net_nfvbench_loop_vn1
    physical_network: data1
    network_type: vlan
    segmentation_id:
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_loop_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

  - network_name: net_nfvbench_loop_vn2
    physical_network: data1
    network_type: vlan
    segmentation_id:
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_loop_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

```

### NFVbench network design - option 4: Virtual Function, 3 physnet

See below the parameters modified for this network design:

```
---
# ... see default file to have other parameters
management_networks:
  - network_name: nfvbexternal_vn1
    external: no
    shared: yes
    cleanup: true

    subnet_name: subnet_nfvbenchexternal_vn1
    gateway: 192.168.90.1
    cidr: 192.168.90.0/24
    enable_dhcp: no
    no_gateway_ip: no

    port_name: nfvbench_mgmt_port
    vnic_type: normal
    security_groups: []
    port_security: no

generator_networks:
  - network_name: net_nfvbench_vn1
    physical_network: data1
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true
    
    topology: loopback_e2e

    subnet_name: subnet_nfvbench_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf1
    vnic_type: direct
    security_groups: []
    port_security: no

   - network_name: net_nfvbench_vn1
    physical_network: data1
    network_type: vlan
    segmentation_id: 2518
    external: yes
    shared: yes
    cleanup: true

    topology: loopback
    
    subnet_name: subnet_nfvbench_vn1
    gateway:
    cidr: 192.168.91.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf3
    vnic_type: direct
    security_groups: []
    port_security: no

  - network_name: net_nfvbench_vn2
    physical_network: data1
    network_type: vlan
    segmentation_id: 2519
    external: yes
    shared: yes
    cleanup: true

    topology: e2e
    
    subnet_name: subnet_nfvbench_vn2
    gateway:
    cidr: 192.168.93.0/24
    enable_dhcp: no
    no_gateway_ip: yes

    port_name: nfvbench_port_vf2
    vnic_type: direct
    security_groups: []
    port_security: no

loop_vm_networks: {{ generator_networks }}
```

## License

Copyright (c) 2021 [Orange, Inc.](https://opensource.orange.com)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
