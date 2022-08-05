---
layout: post
title: Azure - Recover VM with Marketplace Plan
date: 2022-08-05 12:18 +0200
author: hung
categories: [Cloud/DevOps, Azure]
tags: [devops, azure, vm, backup, recovery]
---

# Recover a VM on Azure

## If the VM is deleted accidentally
#### Prerequisites: The snapshots for VM disks must be existing.

- Open Snapshot page on Azure portal UI
- Create a new disk from snapshot
- After new disk is created, open portal UI page of the disk
- Use Create VM option to create new VM
- Fill in the configuration parameters needed to create the VM
- Create the VM
- Associate the old public IP address to new VM
- Drink coffee and test

## If the VM is generalized
#### Prerequisites: The VM and its disk are not deleted.

- Open the VM page on Azure portal UI
- Go to Disks
- Choose the OS Disk
- Create a snapshot from the disk
- After the snapshot is created, open the snapshot
- Create a new disk from snapshot
- After new disk is created, open portal UI page of the disk
- Use Create VM option to create new VM
- Fill in the configuration parameters needed to create the VM
- Create the VM
- Disassociate public IP address from the old VM
- Associate same public IP address to new VM
- Drink coffee and test

## In case the VM is created from Marketplace

We have this error: "Creating a virtual machine from Marketplace image or a custom image sourced from a Marketplace image requires Plan information in the request."

If the old VM still exists, check the JSON view of the VM in Overview tab, there is information about the plan"

```json
{
    ...
    "type": "Microsoft.Compute/virtualMachines",
    "location": "westeurope",
    "plan": {
        "name": "tb-pe-byol",
        "publisher": "things-board",
        "product": "tb-pe-byol"
    },
    ...
}

```

#### Must use ARM template or AZ CLI to create VM:

az cli:

```bash
az vm create --name NewVM --location westeurope --plan-name "tb-pe-byol" --plan-product "tb-pe-byol" --plan-publisher "things-board" -g RESOURCE_GROUP_NAME_HERE --attach-os-disk NAME_OF_OS_DISK_HERE --os-type linux(or windows)
```