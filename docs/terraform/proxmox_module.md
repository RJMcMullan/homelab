# Proxmox VM Provisioning with Terraform

This Terraform configuration automates the provisioning and configuration of virtual machines (VMs) on a Proxmox Virtual Environment (PVE) server. By defining VM specifications in a structured format, you can deploy multiple VMs with cloud-init configurations, custom resource allocations, and QEMU guest agent support.

## Overview

This Terraform setup allows you to create and configure VMs on a Proxmox server by specifying key attributes like CPU, memory, disk storage, network settings, and cloud-init customization. The configuration also supports cloning existing VM templates and configuring VMs with dynamic disk attachments.

### Key Features

- **Customizable VM Specs**: Configure CPU cores, memory, storage, and network settings.
- **Cloud-Init Support**: Customize initialization scripts, SSH keys, and packages via cloud-init.
- **Agent Integration**: Optionally enable QEMU guest agent for enhanced VM functionality.
- **Dynamic Disks**: Attach multiple storage disks dynamically.
- **Detailed Outputs**: Get essential details of the provisioned VMs for further automation.

---

## Requirements

- **Terraform**: Version >= 0.14
- **Proxmox Provider**: Version 0.43.2 from `bpg/proxmox`
- **Proxmox Server**: A running Proxmox VE instance with API access enabled

## Provider Configuration

The following provider configuration is required for this module:

```hcl
terraform {
  required_version = ">= 0.14"
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "0.43.2"
    }
  }
}
```

## Input Variables

The main input variables are defined to allow flexibility in configuring VMs. Here are the primary variables you will configure:

### `gateway_ip`

- **Type**: `string`
- **Description**: The network gateway IP address.
- **Default**: `"192.168.7.1"`

### `vm_os`

- **Type**: `string`
- **Description**: Operating system name for the VMs.

### `pve_vms`

- **Type**: `map(object({...}))`
- **Description**: A map of VM configurations. Each VM entry includes details for CPU, memory, disks, networking, cloud-init, and more.
  
#### Structure:
  - **name**: VM name.
  - **clone_id**: ID of the VM template to clone.
  - **ipv4_address**: IPv4 address to assign to the VM.
  - **memory**: Memory allocation in MB.
  - **cores**: Number of CPU cores.
  - **dns**: List of DNS servers.
  - **agent_enabled**: Enable the QEMU guest agent (boolean).
  - **vga_memory**: VGA memory allocation in MB.
  - **vga_type**: VGA type (e.g., `qxl`, `virtio`).
  - **cloud_init_* properties**: Parameters specific to cloud-init configuration.
  - Additional attributes for startup order, SCSI configuration, disk attachments, and more.

Refer to the `variables.tf` file for a complete list of configurable options within `pve_vms`.

---

## Outputs

The following outputs provide details of the created VMs, which can be utilized in further automation or infrastructure monitoring:

### `vm_names`

- **Description**: List of the VM names as defined in the configuration.
- **Example**:
```json
  ["vm1", "vm2", "vm3"]
```

### Outputs

The following outputs provide details of the created VMs, which can be utilized in further automation or infrastructure monitoring:

### `vm_names`
- Description: List of the VM names as defined in the configuration.

Example:

```json
["vm1", "vm2", "vm3"]
```
### `vm_id`
- Description: Mapping of VM names to their Proxmox VM IDs.

Example:

```json
{
  "vm1": 101,
  "vm2": 102
}
```

### `cloud_init_file_names`
Description: List of cloud-init file names generated for each VM.

Example:
```json
["pve-vm-vm1-cloud-init.yaml", "pve-vm-vm2-cloud-init.yaml"]
```
### `ipv4_addresses`
Description: IPv4 addresses for each VM as provided by the QEMU guest agent.

Example:
```json
{
  "vm1": ["192.168.7.10"],
  "vm2": ["192.168.7.11"]
}
```
Example Usage

Here’s an example of how to use this Terraform configuration to provision VMs:

```hcl
module "proxmox_vms" {
  source = "./proxmox-vm-module"

  gateway_ip = "192.168.7.1"
  pve_vms = {
    vm1 = {
      name             = "example-vm1"
      clone_id         = 100
      ipv4_address     = "192.168.7.10"
      memory           = 2048
      cores            = 2
      dns              = ["8.8.8.8", "8.8.4.4"]
      sockets          = 1
      vm_id            = 101
      node_name        = "pve-node1"
      numa             = false
      agent_enabled    = true
      agent_timeout    = "30s"
      machine          = "q35"
      bios             = "seabios"
      os_type          = "ubuntu"
      scsi_hardware    = "virtio-scsi-pci"
      vga_memory       = 16
      vga_enabled      = true
      vga_type         = "qxl"
      boot_disk_datastore_id = "local-lvm"
      boot_disk_interface    = "scsi0"
      boot_disk_size         = 32
      disks                  = []
      tags                   = ["web", "prod"]
      cloud_init_content_type = "cloud-init"
      cloud_init_datastore_id = "local"
      cloud_init_node_name    = "pve-node1"
      groups                 = ["sudo"]
      sudo_config            = ["ALL=(ALL) NOPASSWD:ALL"]
      packages               = ["htop", "curl"]
      username               = "admin"
      fullname               = "Administrator"
      hostname               = "example-vm1"
      fqdn                   = "example-vm1.local"
      ssh_authorized_keys    = ["ssh-rsa AAAA..."]
    }
  }
}
```

This example demonstrates defining a single VM with custom specifications, cloud-init configuration, and network settings.

<!-- BEGIN_TF_DOCS -->
Terraform project to test proxmox vm creation using a custom proxmox terraform module.

#### Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 0.14 |
| <a name="requirement_proxmox"></a> [proxmox](#requirement\_proxmox) | 0.43.2 |

#### Providers

| Name | Version |
|------|---------|
| <a name="provider_proxmox"></a> [proxmox](#provider\_proxmox) | 0.43.2 |

#### Modules

No modules.

#### Resources

| Name | Type |
|------|------|
| [proxmox_virtual_environment_file.cloud_init](https://registry.terraform.io/providers/bpg/proxmox/0.43.2/docs/resources/virtual_environment_file) | resource |
| [proxmox_virtual_environment_vm.virtual_machine](https://registry.terraform.io/providers/bpg/proxmox/0.43.2/docs/resources/virtual_environment_vm) | resource |

#### Inputs

| Name | Description | Type |
|------|-------------|------|
| <a name="input_vm_os"></a> [vm\_os](#input\_vm\_os) | The vm operating system name | `string` |
| <a name="input_gateway_ip"></a> [gateway\_ip](#input\_gateway\_ip) | The network gateway IP | `string` |
| <a name="input_pve_vms"></a> [pve\_vms](#input\_pve\_vms) | n/a | <pre>map(object({<br>    name                   = string<br>    clone_id               = number<br>    ipv4_address           = string<br>    memory                 = number<br>    cores                  = number<br>    dns                    = list(string)<br>    sockets                = string<br>    vm_id                  = number<br>    node_name              = string<br>    numa                   = bool<br>    agent_enabled          = bool<br>    agent_timeout          = string<br>    machine                = string<br>    bios                   = string<br>    # startup_order          = number<br>    # startup_delay          = number<br>    # startup_down_delay     = number<br>    os_type                = string<br>    scsi_hardware          = string<br>    vga_memory             = number<br>    vga_enabled            = bool<br>    vga_type               = string<br>    network_device_bridge  = string<br>    vm_description         = string<br>    boot_disk_datastore_id = string<br>    boot_disk_interface    = string<br>    boot_disk_size         = number<br>    disks = list(object({<br>      disk_datastore_id = string<br>      disk_file_format  = string<br>      disk_size         = number<br>      disk_interface    = string<br>    }))<br>    tags = list(string)<br>    # Cloud Init<br>    cloud_init_content_type = string<br>    cloud_init_datastore_id = string<br>    cloud_init_node_name    = string<br>    groups                  = list(string)<br>    sudo_config             = string<br>    packages                = list(string)<br>    username                = string<br>    fullname                = string<br>    hostname                = string<br>    fqdn                    = string<br>    # # Cloud Image<br>    # cloud_image_url          = string<br>    # cloud_image_node_name    = string<br>    # cloud_image_datastore_id = string<br>    # cloud_image_content_type = string<br>    <br>    <br>    <br><br>  }))</pre> |
For a complete list of inputs and their descriptions for the Proxmox provider, refer to the [Proxmox Provider Documentation](https://registry.terraform.io/providers/bpg/proxmox/latest/docs).

#### Outputs

| Name | Description |
|------|-------------|
| <a name="output_cloud_init_file_names"></a> [cloud\_init\_file\_names](#output\_cloud\_init\_file\_names) | The cloud init file names |
| <a name="output_ipv4_addresses"></a> [ipv4\_addresses](#output\_ipv4\_addresses) | The IPv4 addresses per network interface published by the QEMU agent (empty list when agent.enabled is false) |
| <a name="output_vm_id"></a> [vm\_id](#output\_vm\_id) | The id of the virtual machine |
| <a name="output_vm_names"></a> [vm\_names](#output\_vm\_names) | The name of each virtual machine |

#### Usage
##### provider.tf
```hcl
terraform {
  required_version = ">= 0.14"
  required_providers {
    proxmox = {
      source  = "bpg/proxmox"
      version = "0.43.2"
    }
  }
}
``` 
##### main.tf
```hcl
#===============================================================================
# Cloud Config (cloud-init)
#===============================================================================

resource "proxmox_virtual_environment_file" "cloud_init" {
  for_each = var.pve_vms

  content_type = each.value.cloud_init_content_type
  datastore_id = each.value.cloud_init_datastore_id
  node_name    = each.value.cloud_init_node_name

  source_raw {
    data = <<EOF
#cloud-config
hostname: ${each.value.hostname}
fqdn: ${each.value.hostname}
manage_etc_hosts: true
locale: en_GB.UTF-8
timezone: Europe/London
chpasswd:
  ## Forcing user to change the default password at first login
  expire: true
users:
  - name: ${each.value.username}
    gecos: ${each.value.fullname}
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: "${join(", ", each.value.groups)}"
    lock_passwd: false
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCRGXMohsqSC9uvxb+BWEACVg5lZGVjhA4i4skD5KGwDK2PgMqLSDOKPiKjTceXC5EXpwWZXtthXC7nfSeAFgM0hJNV0YPDnaVLXlkx2QjOxTyxjEebAOobrwxM0027w3pukTXU9slHzfiwANAbvDwk4eb+jShOAchsOcXyV7a3wHHJrz5oOEIxtkKmPYFyf++EnBEKPes8ETKt9LzKPlle7BQleyapmT1VdWtaDb3G1BzkJJmB53MGIzg4hxbKdvuWATYP14n8kBZT5nLURyVWqQC6h++szXUNJHIs4D4hC17NSAiYwOGQDD+OfPF4K4wpIGQQYdEy9kmrqTbnIU6Q9h1xAbwqq7+oyOAoFkgoUD2JsH1jBCij8uBNpfFiKVqrxm406PiJ/FWZnMxjZF6XFV/nqlttMdHCiZWKaxIaSMtIaZbrbzG7FwEkBWA2gQCKOc7ygLQsHQmlpbMyh2Z4aIAbEcYndkopC9XcEsHS1qcJVRIV4frT2oDO/h8JY2U= rory@rory-Z390-M-GAMING
chpasswd:
  ## Forcing user to change the default password at first login
  expire: true
  list: |
    rory:password
package_update: true
package_upgrade: true
${length(each.value.packages) > 0 ? "packages:\n${join("\n", [for pkg in each.value.packages : "  - ${pkg}"])}" : ""}
runcmd:
  - systemctl enable qemu-guest-agent
  - systemctl start qemu-guest-agent
power_state:
    delay: now
    mode: reboot
    message: Rebooting after cloud-init completion
    condition: true

  EOF

    file_name = "pve-vm-${each.value.name}-cloud-init.yaml"
  }
}

#===============================================================================
# Ubuntu Cloud Image
#===============================================================================

# resource "proxmox_virtual_environment_download_file" "cloud_image" {
#   for_each     = var.pve_vms
  
#   content_type = each.value.cloud_image_content_type
#   datastore_id = each.value.cloud_image_datastore_id
#   node_name    = each.value.cloud_image_node_name
#   url          = each.value.cloud_image_url
# }


# resource "proxmox_virtual_environment_file" "cloud_image" {
#   for_each     = var.pve_vms
#   content_type = each.value.cloud_image_content_type
#   datastore_id = each.value.cloud_init_datastore_id
#   node_name    = each.value.cloud_image_node_name

#   source_file {
#     path = each.value.cloud_image_url
#   }
# }
``` 
##### local.tf
```hcl
resource "proxmox_virtual_environment_vm" "virtual_machine" {
  for_each = var.pve_vms

  name        = each.value.name
  description = each.value.vm_description
  node_name   = each.value.node_name
  migrate     = false // migrate the VM on node change
  vm_id       = each.value.vm_id
  tags        = each.value.tags

  clone {
    vm_id = each.value.clone_id
 }

  bios    = each.value.bios
  machine = each.value.machine

  memory {
    dedicated = each.value.memory
  }

  cpu {
    cores   = each.value.cores
    numa    = each.value.numa
    sockets = each.value.sockets

  }

  serial_device {}

  vga {
    enabled = each.value.vga_enabled
    memory  = each.value.vga_memory
    type    = each.value.vga_type


  }

  agent {
    enabled = each.value.agent_enabled
    timeout = each.value.agent_timeout
  }

  # startup {
  #   order      = each.value.startup_order
  #   up_delay   = each.value.startup_delay
  #   down_delay = each.value.startup_down_delay
  # }

  operating_system {
    type = each.value.os_type
  }

  scsi_hardware = each.value.scsi_hardware
  
  
  initialization {
    ip_config {
      ipv4 {
        address = each.value.ipv4_address
        gateway = var.gateway_ip
        }
      ipv6 {
        address = "dhcp"

      }
    }

    dns {
      servers = each.value.dns
    }

    user_data_file_id = proxmox_virtual_environment_file.cloud_init[each.key].id
  }

  # boot disk
  disk {
    datastore_id = each.value.boot_disk_datastore_id
    interface    = each.value.boot_disk_interface
    size         = each.value.boot_disk_size
  }

  # attached disks from data_vm
  dynamic "disk" {
    for_each = each.value.disks
    content {
      datastore_id = disk.value.disk_datastore_id
      file_format  = disk.value.disk_file_format
      size         = disk.value.disk_size
      # assign from scsi1 and up
      interface    = disk.value.disk_interface


    }

  }
}
```   
##### variables.tf
```hcl
# VM clone
variable "gateway_ip" {
  type        = string
  description = "The network gateway IP"
  default     = "192.168.7.1"
}

variable "vm_os" {
  type        = string
  description = "The vm operating system name"

}

variable "pve_vms" {
  type = map(object({
    name                   = string
    clone_id               = number
    ipv4_address           = string
    memory                 = number
    cores                  = number
    dns                    = list(string)
    sockets                = string
    vm_id                  = number
    node_name              = string
    numa                   = bool
    agent_enabled          = bool
    agent_timeout          = string
    machine                = string
    bios                   = string
    # startup_order          = number
    # startup_delay          = number
    # startup_down_delay     = number
    os_type                = string
    scsi_hardware          = string
    vga_memory             = number
    vga_enabled            = bool
    vga_type               = string
    network_device_bridge  = string
    vm_description         = string
    boot_disk_datastore_id = string
    boot_disk_interface    = string
    boot_disk_size         = number
    disks = list(object({
      disk_datastore_id = string
      disk_file_format  = string
      disk_size         = number
      disk_interface    = string
    }))
    tags = list(string)
    # Cloud Init
    cloud_init_content_type = string
    cloud_init_datastore_id = string
    cloud_init_node_name    = string
    groups                  = list(string)
    sudo_config             = string
    packages                = list(string)
    username                = string
    fullname                = string
    hostname                = string
    fqdn                    = string
    # # Cloud Image
    # cloud_image_url          = string
    # cloud_image_node_name    = string
    # cloud_image_datastore_id = string
    # cloud_image_content_type = string
    
    
    

  }))
  default = {}
}


```  
##### outputs.tf
```hcl
output "vm_names" {
  value       = [for vm_name, vm_config in var.pve_vms : vm_config.name]
  description = "The name of each virtual machine"
}

output "vm_id" {
  value = {
    for vm_name, vm_config in var.pve_vms :
    vm_name => proxmox_virtual_environment_vm.virtual_machine[vm_name].id
  }
  description = "The id of the virtual machine"
}

output "cloud_init_file_names" {
  value       = [for key, cloud_init_resource in proxmox_virtual_environment_file.cloud_init : cloud_init_resource.file_name]
  description = "The cloud init file names"
}


output "ipv4_addresses" {
  value = {
    for vm_name, vm_config in var.pve_vms :
    vm_name => proxmox_virtual_environment_vm.virtual_machine[vm_name].ipv4_addresses
  }
  description = "The IPv4 addresses per network interface published by the QEMU agent (empty list when agent.enabled is false)"
}
```  

<!-- END_TF_DOCS -->

### License

This project is licensed under the MIT License. See the LICENSE file for details.