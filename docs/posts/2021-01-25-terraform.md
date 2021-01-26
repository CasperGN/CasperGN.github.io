---
layout: post
date: 2021-01-25
title: Automatic provisioning with Terraform 
tags: General, Automation
author: CasperGN
---

First I would like to describe my end-goal here. My project [ade](https://github.com/CasperGN/ActiveDirectoryEnumeration) sometimes suffer from much manual testing. I have to either go to work to start pounding the domain controller or connect to my account and VPN setup to HackTheBox and hope I can utilise one of the AD machines in the lab.  
This way of working is extremely ineffective and fairly cumbersome.  
  
And here comes Terraform.  
  
The goal with this learning project is not only to better understand and use Terraform to provision resources in Azure but also to end up with the automation that would be required in order to build an Active Directory domain with a domain controller and 5 domain joined machines. The domain is required to have pre-defined vulnerable configurations to test the tool in the Github Action workflow and hence we also need to restrict access.

*Requirements*:

- Provision 6 Windows Virtual Machines
- Install Active Directory on one
- Remaining 5 VMs must be domain joined and expose an SMB share
- Import or configure the domain to contain the following vulnerabilities:
    - AS-REP Roasting
    - Kerberoasting
    - Anonymous Bind
    - One user/service account with password in attribute
    - Silver Ticket
    - Golden Ticket
    - CPassword in a GPO on SYSVOL
- Add provisioning to Github Workflow
- Ensure machines are always deleted as a last step
- Restrict access to only the Github Runner

<hr />

I've created the repo [here](https://github.com/CasperGN/autoinfra) and will be following the initial guide from [hashicorp](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code?in=terraform/azure-get-started) for Azure since Azure - for now at least - is the most relevant provider in my daily work.

The repo will show how to deploy a single VM containing a marketplace image for Active Directory 2019.

The most interesting part which deviates from the Hashicorp guide is the acceptance of the Marketplace agreement

```terraform
# main.tf
resource "azurerm_marketplace_agreement" "cloud-infrastructure-services" {
  publisher = var.dc_settings["publisher"]
  offer     = var.dc_settings["offer"]
  plan      = "hourly"
}
```

As well as the configuration of the VM resource itself

```terraform
# instance.tf
resource "azurerm_virtual_machine" "vm" {
  name                  = "${var.prefix}-VM"
  location              = var.location
  resource_group_name   = azurerm_resource_group.rg.name
  network_interface_ids = [azurerm_network_interface.nic.id]
  vm_size               = var.dc_settings["vm_size"]


  storage_os_disk {
    name          = "${var.prefix}-OsDisk"
    caching       = "ReadWrite"
    create_option = "FromImage"
  }

  storage_image_reference {
    publisher = var.dc_settings["publisher"]
    offer     = var.dc_settings["offer"]
    sku       = var.dc_settings["sku"]
    version   = var.dc_settings["version"]
  }

  os_profile {
    computer_name  = "${var.prefix}-VM"
    admin_username = var.admin_username
    admin_password = var.admin_password
  }

  os_profile_windows_config {}

  plan {
    name = var.dc_settings["offer"]
    publisher = var.dc_settings["publisher"]
    product = var.dc_settings["offer"]
  }
}
```

Notice the `plan` block, which is required when using Marketplace items see [ref1](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_machine#plan) and [ref2](https://docs.microsoft.com/en-us/azure/virtual-machines/marketplace-images) for more insight. 

Executing the scripts took me about 7 minutes with `vm_size = "Standard_A1_v2"` to complete.

*Disclaimer*: The NSG security rules found in `network.tf` is horrible and needs to be fixed to specific port openings. This will be handled at a later stage.

Next step will be to configure the domain. And here I'll put my trust in an old friend, `Ansible`.

Laters,  
CGN

[Return](../)