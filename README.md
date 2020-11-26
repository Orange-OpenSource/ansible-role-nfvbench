# Ansible role: nfvbench

This repo hosts the `nfvbench` Ansible role.

This role aims to deploy [NFVbench](https://github.com/opnfv/nfvbench) tool inside an SR-IOV VM on a Openstack platform.
The role includes tasks using the `openstack.cloud` Ansible Collection.

## Installation and Usage

### Installing dependencies

For using the NFVbench Ansible role firstly you need to install `ansible` and `openstacksdk` Python modules on your Ansible controller.
For example with pip:

```bash
pip install ansible openstacksdk
```

OpenStackSDK has to be available to Ansible and to the Python interpreter on the host, where Ansible executes the module (target host).
Please note, that under some circumstances Ansible might invoke a non-standard Python interpreter on the target host.
Using Python version 3 is required for OpenstackSDK.

---

#### NOTE

OpenstackSDK is better to be the last stable version. It should NOT be installed on Openstack nodes,
but rather on operators host (aka "Ansible controller"). OpenstackSDK from last version supports
operations on all Openstack cloud versions. Therefore OpenstackSDK module version doesn't have to match
Openstack cloud version usually.

---

### Installing the Openstack Cloud Collection from Ansible Galaxy

Before using this ansible role you need to install the Openstack Cloud collection with the `ansible-galaxy` CLI:

`ansible-galaxy collection install -r requirements.yml`

### Installing the NFVbench ansible role from Ansible Galaxy

Before using this ansible role you need to install the Openstack Cloud collection with the `ansible-galaxy` CLI:

`ansible-galaxy install frmenguy.nfvbench`


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
    - role: frmenguy.nfvbench

```

## Examples of nfvbench ansible role configuration

Default configuration is available in `defaults/main.yml` file in nfvbench ansible role sources.

This configuration can be overridden by replacing this file or using Ansible `extra_vars`.

`Note: Please find network design options details here :` [NFVbench network design options](docs/nfvbenchvm-network-design.md)

#### NFVbench network design - option 1: Physical Function, 3 physnets

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

#### NFVbench network design - option 2: Physical Function, 1 physnet

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

#### NFVbench network design - option 3: Virtual Function, 3 physnets

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

#### NFVbench network design - option 4: Virtual Function, 3 physnet

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