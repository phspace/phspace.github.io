---
layout: post
title: Upgrade Dell Server Firmware with Ansible
date: 2023-05-03 00:00 +0200
author: hung
categories: [Cloud/DevOps, Ansible]
tags: [devops, ansible, dell, hardware, server]
---

## IDRAC VS REDFISH

Redfish and iDRAC are both technologies used for remote server management, but they have some differences in terms of features and implementation.

Redfish is an open, standard API (Application Programming Interface) for server management that is supported by many hardware vendors. It provides a unified, modern interface for managing servers, storage, and networking equipment, and is designed to be easily extensible and scalable. Redfish can be used for monitoring and controlling hardware components, setting up firmware configurations, managing power consumption, and more.

iDRAC (Integrated Dell Remote Access Controller) is a proprietary technology developed by Dell for remote server management. It provides a web-based interface that allows users to monitor and manage various aspects of the server, including hardware configuration, power consumption, and firmware updates. iDRAC also includes features such as remote console access, virtual media support, and remote scripting capabilities.

One key difference between Redfish and iDRAC is that Redfish is an open standard, while iDRAC is a proprietary technology. This means that Redfish can be used with hardware from multiple vendors, while iDRAC is specific to Dell hardware. Additionally, Redfish is designed to be extensible and can be customized by vendors to meet their specific needs, while iDRAC is a more closed system.

Overall, both Redfish and iDRAC provide powerful tools for remote server management, and the choice between them will depend on the specific needs and hardware environment of the user.

## IDRAC

To boot your Dell PowerEdge R230 server from a bootable ISO file stored on an iPXE server using Ansible and iDRAC, follow these steps:

1. Install required tools and dependencies:

You'll need Ansible and the OpenManage Ansible Modules for iDRAC installed on your management machine.

To install Ansible, follow the official installation guide for your operating system: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

To install OpenManage Ansible Modules for iDRAC, run the following command:

```bash
pip install omsdk
```

Next, download the Dell EMC OpenManage Ansible Modules from GitHub:

```bash
git clone https://github.com/dell/dellemc-openmanage-ansible-modules.git
```

Navigate to the downloaded directory and install the modules:

```bash
cd dellemc-openmanage-ansible-modules
python setup.py install
```

2. Prepare the firmware update files:

Download the required firmware update files from the Dell support site and place them in a directory accessible by the PXE boot server. Make sure the PXE boot server is configured to serve the firmware files.

3. Create an Ansible playbook:

Create an Ansible playbook (e.g., boot_from_ipxe_iso.yml) to boot the server using the dellemc modules:

```yaml
---
- name: Boot Dell Server from iPXE ISO
  hosts: servers
  gather_facts: no
  tasks:
    - name: Mount the ISO from the iPXE server
      dellemc.openmanage.dellemc_idrac_server_config_profile:
        idrac_ip: "{{ inventory_hostname }}"
        idrac_user: "{{ idrac_user }}"
        idrac_password: "{{ idrac_password }}"
        share_name: "http://{{ ipxe_server }}/path/to/bootable/iso/filename.iso"
        scp_action: "Import"
        scp_components: "SystemConfiguration"
        job_wait: True

    - name: Change the boot order to boot from virtual media
      dellemc.openmanage.dellemc_configure_bios:
        idrac_ip: "{{ inventory_hostname }}"
        idrac_user: "{{ idrac_user }}"
        idrac_password: "{{ idrac_password }}"
        attributes:
          BootMode: "Bios"
          BootSeqRetry: "Enabled"
          BootSeq: "IPLoM.1,HardDisk.List.1-1,Optical.SATAEmbedded.J-1"
        job_wait: True

    - name: Reboot the server
      dellemc.openmanage.redfish_command:
        category: Systems
        command: PowerGracefulRestart
        baseuri: "{{ inventory_hostname }}"
        username: "{{ idrac_user }}"
        password: "{{ idrac_password }}"
```

This playbook mounts the ISO from the iPXE server, sets the boot order to boot from virtual media, and reboots the server.

4. Create an inventory file:

Create an inventory file (e.g., inventory.ini) that lists your server's iDRAC IP addresses:

```
[servers]
192.168.1.100
```

5. Run the Ansible playbook:

Execute the Ansible playbook with the appropriate variables:

```bash
ansible-playbook -i inventory.ini boot_from_ipxe_iso.yml --extra-vars "idrac_user=root idrac_password=your_idrac_password ipxe_server=your_ipxe_server_ip"
```

This playbook will configure your server to boot from the ISO file stored on the iPXE server by connecting to the iDRAC and applying the necessary settings. The server will reboot as part of the process.

## REDFISH

To update the firmware of your Dell PowerEdge R230 server using Redfish API with Ansible, follow these steps:

1. Install required dependencies:

You'll need Ansible and the Redfish Ansible Modules installed on your management machine.

To install Ansible, follow the official installation guide for your operating system: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

To install the Redfish Ansible Modules, run the following command:

```bash
pip install ansible-modules-redfish
```

2. Prepare the firmware update files:

Download the required firmware update files from the Dell support site and place them in a directory accessible by an HTTP server. Make sure the HTTP server is configured to serve the firmware files.

3. Create an Ansible playbook:

Create an Ansible playbook (e.g., update_firmware_redfish.yml) to update the firmware using the redfish modules:

```yaml
---
- name: Boot Dell Server from iPXE ISO using Redfish
  hosts: servers
  gather_facts: no
  tasks:
    - name: Mount the ISO from the iPXE server
      redfish_command:
        category: Manager
        command: MountVirtualMedia
        baseuri: "{{ inventory_hostname }}"
        username: "{{ redfish_user }}"
        password: "{{ redfish_password }}"
        media_url: "http://{{ ipxe_server }}/path/to/bootable/iso/filename.iso"
        media_type: CD

    - name: Set the server to boot from virtual media
      redfish_config:
        category: Systems
        command: SetOneTimeBoot
        boot_device: UefiTarget
        baseuri: "{{ inventory_hostname }}"
        username: "{{ redfish_user }}"
        password: "{{ redfish_password }}"

    - name: Reboot the server
      redfish_command:
        category: Systems
        command: PowerGracefulRestart
        baseuri: "{{ inventory_hostname }}"
        username: "{{ redfish_user }}"
        password: "{{ redfish_password }}"
```

This playbook mounts the ISO from the iPXE server, sets the boot order to boot from virtual media, and reboots the server.

4. Create an inventory file:

Create an inventory file (e.g., inventory.ini) that lists your server's iDRAC IP addresses:

```
[servers]
192.168.1.100
```

5. Run the Ansible playbook:

Execute the Ansible playbook with the appropriate variables:

```bash
ansible-playbook -i inventory.ini update_firmware_redfish.yml --extra-vars "redfish_user=root redfish_password=your_redfish_password http_server=your_http_server_ip"
```

This playbook will configure your server to boot from the ISO file stored on the iPXE server by connecting to the iDRAC using Redfish API and applying the necessary settings. The server will reboot as part of the process.