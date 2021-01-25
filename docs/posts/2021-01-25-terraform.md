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

Next step will be to configure the domain. And here I'll put my trust in an old friend, `Ansible`.

Laters,  
CGN

[Return](../)