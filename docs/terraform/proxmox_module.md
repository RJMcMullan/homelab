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
##### cloud-init.tf
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
    sudo: "${join(", ", each.value.sudo_config)}"
    groups: "${join(", ", each.value.groups)}"
    lock_passwd: false
    ssh_authorized_keys:
    %{ for key in each.value.ssh_authorized_keys ~}
      - ${key}
    %{ endfor ~}
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
``` 
#### vm.tf

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
      interface = disk.value.disk_interface


    }

  }
}
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