---
title: Ansible oVirt modules
category: feature
authors: omachace
feature_name: Ansible oVirt modules
feature_status: Released
---

### Summary

The goal is to have a module for every entity oVirt has, so users can manage whole oVirt environment via Ansible playbooks.

### Owner

*   Name: [Ondra Machacek](User:omachace)
*   Email: <omachace@redhat.com>

# Ansible

## About

Ansible is an IT automation tool. It can configure systems, deploy software, and orchestrate more advanced IT tasks such
as continuous deployments or zero downtime rolling updates.

Ansible’s main goals are simplicity and ease-of-use. It also has a strong focus on security and reliability,
featuring a minimum of moving parts, usage of OpenSSH for transport (with an accelerated socket mode and pull modes
as alternatives), and a language that is designed around auditability by humans–even those not familiar with the program.

## Playbooks
Playbooks are Ansible’s configuration, deployment, and orchestration language. They can describe a policy you want your remote
systems to enforce, or a set of steps in a general IT process. Playbooks uses [YAML](http://yaml.org) as support language. In
general playbooks consist of tasks. Every tasks executes specific module with parameters.

## Modules
Ansible ships with a number of modules (called the ‘module library’) that can be executed directly on remote hosts or through 
Playbooks. Modules have to be idempotent. The concept that change commands should only be applied when they need to be applied,
and that it is better to describe the desired state of a system than the process of how to get to that state. This feature page
describes oVirt ansible modules.

# Ansible oVirt modules

So far few modules has been merged to Ansible and are part of Ansible extras [modules](https://github.com/ansible/ansible-modules-extras/).

__Important__: Ansible oVirt modules works with oVirt version 4 and higher.

__Note__: Links to modules documentation which will be in Ansible 2.3 are temporary on readthedocs page
once, the modules will be merged and Ansible 2.3 will be release we will change links to official documentation.

# Ansible 2.2
Following modules has been merged and can be used in Ansible version 2.2.

The following table shows oVirt modules and version of Ansible where the modules are shipped:

| Module                     | Version |
| -------------------------- | ------- |
| [ovirt_auth]               | 2.2     |
| [ovirt_vms]                | 2.2     |
| [ovirt_disks]              | 2.2     |
| [ovirt_datacenters]        | 2.3     |
| [ovirt_clusters]           | 2.3     |
| [ovirt_networks]           | 2.3     |
| [ovirt_storage_domains]    | 2.3     |
| [ovirt_hosts]              | 2.3     |
| [ovirt_host_pm]            | 2.3     |
| [ovirt_host_networks]      | 2.3     |
| [ovirt_external_providers] | 2.3     |
| [ovirt_nics]               | 2.3     |
| [ovirt_templates]          | 2.3     |
| [ovirt_vmpools]            | 2.3     |
| [ovirt_users]              | 2.3     |
| [ovirt_groups]             | 2.3     |
| [ovirt_permissions]        | 2.3     |

[ovirt_auth]: #ovirt_auth
[ovirt_vms]: #ovirt_vms
[ovirt_disks]: #ovirt_disks
[ovirt_datacenters]: #ovirt_datacenters
[ovirt_clusters]: #ovirt_clusters
[ovirt_networks]: #ovirt_networks
[ovirt_storage_domains]: #ovirt_storage_domains
[ovirt_hosts]: #ovirt_hosts
[ovirt_host_pm]: #ovirt_host_pm
[ovirt_host_networks]: #ovirt_host_networks
[ovirt_external_providers]: #ovirt_external_providers
[ovirt_nics]: #ovirt_nics
[ovirt_templates]: #ovirt_templates
[ovirt_vmpools]: #ovirt_vmpools
[ovirt_users]: #ovirt_users
[ovirt_groups]: #ovirt_groups
[ovirt_permissions]: #ovirt_permissions

## ovirt_auth
[ovirt_auth](http://docs.ansible.com/ansible/ovirt_auth_module.html)
[[source]](https://github.com/ansible/ansible-modules-extras/blob/devel/cloud/ovirt/ovirt_auth.py)
module authenticates to oVirt engine and creates SSO
token, which should be later used in all other oVirt modules, so all modules don’t need to perform login and logout. This
module returns an Ansible fact called _ovirt_auth_. Every module can use this fact as auth parameter, to perform authentication

### Example
```yaml
# Obtain SSO token with using username/password credentials
no_log: true
ovirt_auth:
  url: https://ovirt.example.com/ovirt-engine/api
  username: admin@internal
  password: 123456
  ca_file: ca.pem

# Revoke the SSO token returned from previous task
ovirt_auth:
  state: absent
  ovirt_auth: "{{ ovirt_auth }}"
```

## ovirt_vms
[ovirt_vms](http://docs.ansible.com/ansible/ovirt_vms_module.html)
[[source]](https://github.com/ansible/ansible-modules-extras/blob/devel/cloud/ovirt/ovirt_vms.py)
module manages whole lifecycle of the Virtual Machine (VM)
in oVirt. In addtion you can add disks and network interfaces to VM from this module, but if you need to manage disks and
network interfaces in more depth you should use modules specifically for it.

### Example
```yaml
# Creates a new Virtual Machine from template named 'rhel7_template'
ovirt_vms:
    state: present
    name: myvm
    template: rhel7_template

# Run VM with cloud init:
ovirt_vms:
    name: rhel7
    template: rhel7
    cluster: Default
    memory: 1GiB
    high_availability: true
    cloud_init:
      nic_boot_protocol: static
      nic_ip_address: 10.34.60.86
      nic_netmask: 255.255.252.0
      nic_gateway: 10.34.63.254
      nic_name: eth1
      nic_on_boot: true
      host_name: example.com
      custom_script: |
        write_files:
         - content: |
             Hello, world!
           path: /tmp/greeting.txt
           permissions: '0644'
      user_name: root
      root_password: super_password

# Migrate/Run VM to/on host named 'host1'
ovirt_vms:
    state: running
    name: myvm
    host: host1

# Change Vm's CD:
ovirt_vms:
    name: myvm
    cd_iso: drivers.iso

# Stop vm:
ovirt_vms:
    state: stopped
    name: myvm

# Hot plug memory to already created and running VM:
# (VM won't be restarted)
ovirt_vms:
    name: myvm
    memory: 4GiB
  
# When change on the VM needs restart of the VM, use next_run state,
# The VM will be updated and rebooted if there are any changes.
# If present state would be used, VM won't be restarted.
ovirt_vms:
    state: next_run
    name: myvm
    boot_devices:
      - network
```

## ovirt_disks
[ovirt_disks](http://docs.ansible.com/ansible/ovirt_disks_module.html)
[[source]](https://github.com/ansible/ansible-modules-extras/blob/devel/cloud/ovirt/ovirt_disks.py)
module to manage Virtual Machine and floating disks in oVirt. This module can attach/detach disks from VM, update attached
disks attributes. This module also handle work with logical units.

### Examples
```yaml
# Create and attach new disk to VM
- ovirt_disks:
    name: myvm_disk
    vm_name: rhel7
    size: 10GiB
    format: cow
    interface: virtio

# Attach logical unit to VM rhel7
- ovirt_disks:
    vm_name: rhel7
    logical_unit:
      target: iqn.2016-08-09.brq.str-01:omachace
      id: 1IET_000d0001
      address: 10.34.63.204
    interface: virtio

# Detach disk from VM
- ovirt_disks:
    state: detached
    name: myvm_disk
    vm_name: rhel7
    size: 10GiB
    format: cow
    interface: virtio
```

# Ansible 2.3
Following modules are pull request ready and will be included in Ansible version 2.3.

## ovirt_datacenters
[ovirt_datacenters](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_datacenters_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_datacenters.py)
module to manage oVirt datacenters. This module can handle create, update and delete action with various parameters on oVirt
datacenter.

### Examples
```yaml
# Create datacenter
- ovirt_datacenters:
    name: mydatacenter
    local: True
    compatibility_version: 4.0
    quota_mode: enabled

# Remove datacenter
- ovirt_datacenters:
    state: absent
    name: mydatacenter
```

## ovirt_clusters
[ovirt_clusters](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_clusters_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_clusters.py)
module to manage oVirt clusters. This module can handle create, update and delete action with various parameters on oVirt
cluster.

### Examples
```yaml
# Create cluster
- ovirt_clusters:
    name: mycluster
    datacenter_name: mydatacenter
    cpu_type: Intel SandyBridge Family
    compatibility_version: 4.0

# Remove cluster
- ovirt_clusters:
    state: absent
    name: mycluster
```

## ovirt_networks
[ovirt_clusters](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_clusters_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_clusters.py)
module to manage oVirt logical datacenter networks. This module can handle create, update and delete action with various
parameters on oVirt logical datacenter networks.

### Examples
```yaml
# Create network
- ovirt_networks:
    datacenter_name: mydatacenter
    name: mynetwork
    vlan_tag: 1
    vm_network: true

# Remove network
- ovirt_networks:
    state: absent
    name: mynetwork
```

## ovirt_storage_domains
[ovirt_storage_domains](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_storage_domains_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_storage_domains.py)
module to manage oVirt storage domains. This module can handle present, absent and maintanence state of the storage domain
with various parameters. The supported storage domains types are _nfs_, _iscsi_, _posixfs_, _glusterfs_ and _fcp_. User can
also handle importing of export/iso storage domain.

### Examples
```yaml
# Add data NFS storage domain
- ovirt_storage_domains:
    name: data_nfs
    host: myhost
    data_center: mydatacenter
    nfs:
      address: 10.34.63.199
      path: /path/data

# Add data iSCSI storage domain:
- ovirt_storage_domains:
    name: data_iscsi
    host: myhost
    data_center: mydatacenter
    iscsi:
      target: iqn.2016-08-09.domain-01:nickname
      lun_id: 1IET_000d0002
      address: 10.34.63.204

# Import export NFS storage domain:
- ovirt_storage_domains:
    domain_function: export
    host: myhost
    data_center: mydatacenter
    nfs:
      address: 10.34.63.199
      path: /path/export

# Remove storage domain
- ovirt_storage_domains:
    state: absent
    name: mystorage_domain
    format: true
```

## ovirt_hosts
[ovirt_hosts](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_hosts_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_hosts.py)
module to manage oVirt hosts. This module can handle _present_, _absent_, _upgraded_ and _maintanence_ state of the host with
various parameters.

### Examples
```yaml
# Add host with username/password
- ovirt_hosts:
    cluster: Default
    name: myhost
    address: 10.34.61.145
    password: secret

# Add host using public key
- ovirt_hosts:
    public_key: true
    cluster: Default
    name: myhost2
    address: 10.34.61.145

# Switch host into maintenance mode:
- ovirt_hosts:
    state: maintenance
    name: myhost

# Upgrade host:
- ovirt_hosts:
    state: upgraded
    name: myhost

# Remove host:
- ovirt_hosts:
    state: absent
    name: myhost
    force: true
```

## ovirt_host_pm
[ovirt_host_pm](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_host_pm_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_host_pm.py)
module to manage oVirt host power management. This module can handle create, update and delete action with various
parameters on oVirt host power management.

### Examples
```yaml
# Add fence agent to host 'myhost'
- ovirt_host_pm:
    name: myhost
    address: 1.2.3.4
    options:
      myoption1: x
      myoption2: y
    username: admin
    password: admin
    type: ipmilan

# Remove ipmilan fence agent with address 1.2.3.4 on host 'myhost'
- ovirt_host_pm:
    state: absent
    name: myhost
    address: 1.2.3.4
    type: ipmilan
```

## ovirt_host_networks
[ovirt_host_networks](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_host_networks_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_host_networks.py)
module to manage oVirt host networks. This module can create/remove bonds on host interfaces and manage logical networks,
labels and vlans above them.

### Examples
```yaml
# Create bond on eth0 and eth1 interface, and put 'myvlan' network on top of it:
- ovirt_host_networks:
    name: myhost
    bond:
      name: bond0
      mode: 2
      interfaces:
        - eth0
        - eth1
    network: myvlan

# Assign network label to host interface
- ovirt_host_networks:
    name: myhost
    interface: eth0
    labels:
      - network_label1

# Assign network to host interface
- ovirt_host_networks:
    name: myhost
    interface: eth0
    network: ovirtmgmt

# Detach network from host
- ovirt_host_networks:
    state: absent
    name: myhost
    network: myvlan
```

## ovirt_external_providers
[ovirt_external_providers](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_external_providers_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_external_providers.py)
module to manage oVirt external providers. This module can handle create, update and delete action with various parameters on
oVirt external providers. Supported external providers are _OpenStackImageProvider_, _OpenStackNetworkProvider_,
_OpenStackVolumeProvider_ and _ExternalHostProvider_.

### Examples
```yaml
# Add image external provider:
- ovirt_external_providers:
    name: image_provider
    type: os_image
    url: http://10.34.63.71:9292
    username: admin
    password: 123456
    tenant: admin
    auth_url: http://10.34.63.71:35357/v2.0/

# Remove image external provider:
- ovirt_external_providers:
    state: absent
    name: image_provider
    type: os_image
```

## ovirt_nics
[ovirt_nics](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_nics_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_nics.py)
module to manage oVirt virtual machines network interfaces. This module can handle _present_, _absent_, _plugged_ and
_unplugged_ state of the network interface with various parameters.

### Examples
```yaml
# Add NIC to VM
- ovirt_nics:
    state: present
    vm_name: myvm
    name: mynic
    interface: e1000
    mac_address: 00:1a:4a:16:01:56
    profile: ovirtmgmt

# Plug NIC to VM
- ovirt_nics:
    state: plugged
    vm_name: myvm
    name: mynic

# Unplug NIC from VM
- ovirt_nics:
    state: unplugged
    vm_name: myvm
    name: mynic

# Remove NIC from VM
- ovirt_nics:
    state: absent
    vm_name: myvm
    name: mynic
```

## ovirt_templates
[ovirt_templates](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_templates_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_templates.py)
module to manage oVirt templates.  This module can handle _present_, _absent_, _imported_ and _exported_ state of the template with various parameters.

### Examples
```yaml
# Create template from VM
- ovirt_templates:
    cluster: Default
    name: mytemplate
    vm_name: rhel7
    cpu_profile: Default
    description: Test

# Import template
- ovirt_templates:
  state: imported
  name: mytemplate
  export_domain: myexport
  storage_domain: mystorage
  cluster: mycluster

# Remove template
- ovirt_templates:
    state: absent
    name: mytemplate
```

## ovirt_vmpools
[ovirt_vmpools](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_vmpools_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_vmpools.py)
module to manage oVirt templates.  This module can handle _present_, _absent_ state of the vmpool with various parameters.

### Examples
```yaml
# Create vm pool from template
- ovirt_vmpools:
    cluster: Default
    name: myvmpool
    template: rhel7
    vm_count: 2
    prestarted: 2
    vm_per_user: 1

# Remove vmpool
- ovirt_vmpools:
    state: absent
    name: myvmpool
    force: true
```

## ovirt_users
[ovirt_users](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_users_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_users.py)
module to manage oVirt users. This module can handle create and delete action with various parameters on oVirt users.

### Examples
```yaml
# Add user user1 from authorization provider example.com-authz
ovirt_users:
    name: user1
    domain: example.com-authz

# Add user user1 from authorization provider example.com-authz
# In case of Active Directory specify UPN:
ovirt_users:
    name: user1@ad2.example.com
    domain: example.com-authz

# Remove user user1 with authorization provider example.com-authz
ovirt_users:
    state: absent
    name: user1
    domain: example.com-authz
```

## ovirt_groups
[ovirt_groups](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_groups_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_groups.py)
module to manage oVirt groups. This module can handle create and delete action with various parameters on oVirt groups.

### Examples
```yaml
# Add group group1 from authorization provider example.com-authz
ovirt_groups:
    name: group1
    domain: example.com-authz

# Add group group1 from authorization provider example.com-authz
# In case of multi-domain Active Directory setup, you should pass
# also namespace, so it adds correct group:
ovirt_groups:
    name: group1
    namespace: dc=ad2,dc=example,dc=com
    domain: example.com-authz

# Remove group group1 with authorization provider example.com-authz
ovirt_groups:
    state: absent
    name: group1
    domain: example.com-authz
```

## ovirt_permissions
[ovirt_permissions](http://ovirt-ansible-modules.readthedocs.io/en/latest/_modules/ovirt_permissions_module.html)
[[source]](https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_permissions.py)
module to manage oVirt permissions. This module can handle assigning and removing of permissions to oVirt entities.

### Examples
```yaml
# Add user user1 from authorization provider example.com-authz
- ovirt_permissions:
    user_name: user1
    authz_name: example.com-authz
    object_type: virtual_machine
    object_name: myvm
    role: UserVmManager

# Remove permission from user
- ovirt_permissions:
    state: absent
    user_name: user1
    authz_name: example.com-authz
    object_type: cluster
    object_name: mycluster
    role: ClusterAdmin
```

# Playbook execution example

### Workspace
First we need to create directory where we will store our playbooks and inventory.

```bash
$ mkdir $HOME/ovirt-ansible
```

### Additional modules
In this example we will use all modules which will be part of the Ansible 2.3, which is not yet realeased, so please first
download all releveant modules and put them into _library_ directory.

```bash
$ mkdir $HOME/ovirt-ansible/library
$ wget https://github.com/machacekondra/ovirt-ansible-example/blob/master/library/ovirt_clusters.py
...
```

### Hosts inventory
Now we need to create a hosts _invetory_ file where we will define a connection details and variables.

```bash
cat >> $HOME/ovirt-ansible/inventory/hosts << EOF
[ovirt]
ovirt.example.com

[ovirt:vars]
username=admin@internal
url=https://ovirt.example.com/ovirt-engine/api
ca_file=/etc/pki/ovirt-engine/ca.pem
external_provider=myprovider
datacenter=mydatacenter
cluster=mycluster
host=myhost
host_address=10.34.60.215
data_name=data
iscsi_name=data2
export_name=export
iso_name=iso
template=rhel7
vm=rhel7
EOF
```

### Playbook directory
Now we need to create symlink to our library from playbooks direcotry, so the ansible knows where to find our additional
modules.

```bash
$ mkdir $HOME/ovirt-ansible/playbooks
$ ln -s $HOME/ovirt-ansible/playbooks/library $HOME/ovirt-ansible/library
```

### Vault
Create vault with oVirt user password, so we don't use this password in plaintext. There is tool which make it easy for
your, just enter this command:

```bash
$ ansible-vault create  ovirt_password.yml
```

This will fire up your editor. Create there _password_ variable with password of your _admin@internal_ user:

```yaml
password: MySuperPasswordOfAdminAtInternal
```

Next it will ask your for a vault password and then it creates _ovirt_password.yml_ file, with your vault.

### Playbook creation
Create a playbook, with tasks you want to execute:
```yaml
cat >> playbooks/setup_demo.yml << EOF
---
- name: Setup oVirt environment
  hosts: ovirt
  tasks:
    - block:
        - name: Include oVirt password
          no_log: true
          include_vars: ovirt_password.yml

        - name: Obtain SSO token
          no_log: false
          ovirt_auth:
            url: "{{ url }}"
            username: "{{ username }}"
            password: "{{ password }}"
            ca_file: "{{ ca_file }}"

        - name: Create datacenter
          ovirt_datacenters:
            auth: "{{ ovirt_auth }}"
            name: "{{ datacenter }}"
            description: mydatacenter
            local: false
            compatibility_version: 4.0
            quota_mode: disabled

        - name: Create cluster
          ovirt_clusters:
            auth: "{{ ovirt_auth }}"
            datacenter_name: "{{ datacenter }}"
            name: "{{ cluster }}"
            cpu_type: Intel Nehalem Family
            description: mycluster
            compatibility_version: 4.0

        - name: Add host using public key
          ovirt_hosts:
            auth: "{{ ovirt_auth }}"
            public_key: true
            cluster: "{{ cluster }}"
            name: "{{ host }}"
            address: "{{ host_address }}"

        - name: Add data NFS storage domain
          ovirt_storage_domains:
            auth: "{{ ovirt_auth }}"
            name: "{{ data_name }}"
            host: "{{ host }}"
            data_center: "{{ datacenter }}"
            nfs:
              address: 10.34.63.199
              path: /omachace/data

        - name: Add data iSCSI storage domain
          ovirt_storage_domains:
            auth: "{{ ovirt_auth }}"
            name: "{{ iscsi_name }}"
            host: "{{ host }}"
            data_center: "{{ datacenter }}"
            iscsi:
              target: iqn.2016-08-09.brq.str-01:omachace
              lun_id: 1IET_000d0002
              address: 10.34.63.204
          ignore_errors: true

        - name: Import export NFS storage domain
          ovirt_storage_domains:
            auth: "{{ ovirt_auth }}"
            name: "{{ export_name }}"
            host: "{{ host }}"
            domain_function: export
            data_center: "{{ datacenter }}"
            nfs:
              address: 10.34.63.199
              path: /omachace/export

        - name: Create ISO NFS storage domain
          ovirt_storage_domains:
            auth: "{{ ovirt_auth }}"
            name: "{{ iso_name }}"
            host: "{{ host }}"
            domain_function: iso
            data_center: "{{ datacenter }}"
            nfs:
              address: 10.34.63.199
              path: /omachace/iso

        - name: Add image external provider
          ovirt_external_providers:
            auth: "{{ ovirt_auth }}"
            name: "{{ external_provider }}"
            type: os_image
            url: http://10.34.63.71:9292
            username: admin
            password: qum5net
            tenant: admin
            auth_url: http://10.34.63.71:35357/v2.0/

        - name: Import template
          ovirt_templates:
            auth: "{{ ovirt_auth }}"
            name: "{{ template }}"
            state: imported
            export_domain: "{{ export_name }}"
            storage_domain: "{{ data_name }}"
            cluster: "{{ cluster }}"

        - name: Create and run VM from template
          ovirt_vms:
            auth: "{{ ovirt_auth }}"
            name: "{{ vm }}"
            template: "{{ template }}"
            cluster: "{{ cluster }}"
            memory: 1GiB
            high_availability: true
            cloud_init:
              host_name: mydomain.local
              custom_script: |
                write_files:
                 - content: |
                     Hello, world!
                   path: /tmp/greeting.txt
                   permissions: '0644'
              user_name: root
              root_password: '1234567'

      always:
        - name: Revoke the SSO token
          ovirt_auth:
            state: absent
            ovirt_auth: "{{ ovirt_auth }}"
EOF
```

### Playbook execution
To execute the playbook run following command:
```bash
$ ansible-playbook -i $HOME/ovirt-ansible/inventory/hosts $HOME/ovirt-ansible/playbooks/setup_demo.yml --ask-vault-pass
Vault password: 

PLAY [Setup oVirt environment] *************************************************

TASK [setup] *******************************************************************
ok: [localhost]

TASK [Include oVirt password] **************************************************
ok: [localhost]
.....
```
It will ask you for the password of the vault and then execute the playbook. Now try to re-run the playbook, and see that no
changes was done on environment.
