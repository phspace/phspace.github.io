---
layout: post
title: Upgrade Dell Server Firmware with Ansible
date: 2023-05-03 00:00 +0200
author: hung
categories: [Cloud/DevOps, Ansible]
tags: [devops, ansible, dell, hardware, server]
---

## IDRAC

To update the firmware of your Dell PowerEdge R230 server with iDRAC 8 using Ansible and PXE boot, follow these steps:

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

2. Configure your iDRAC for PXE boot:

Access the iDRAC web interface, go to the "iDRAC Settings" page, and configure the server to boot from the network. Make sure to save the changes.

3. Prepare the firmware update files:

Download the required firmware update files from the Dell support site and place them in a directory accessible by the PXE boot server. Make sure the PXE boot server is configured to serve the firmware files.

4. Create an Ansible playbook:

Create an Ansible playbook (e.g., update_firmware.yml) to update the firmware using the dellemc modules:

```yaml
---
- name: Update Dell Server Firmware
  hosts: servers
  gather_facts: no
  tasks:
    - name: Update firmware from a repository on a network share (PXE)
      dellemc.openmanage.dellemc_idrac_firmware:
        idrac_ip: "{{ inventory_hostname }}"
        idrac_user: "{{ idrac_user }}"
        idrac_password: "{{ idrac_password }}"
        share_name: "{{ pxe_boot_server }}/path/to/firmware/files"
        share_user: "{{ share_user }}"
        share_password: "{{ share_password }}"
        reboot: True
        job_wait: True
```

5. Create an inventory file:

Create an inventory file (e.g., inventory.ini) that lists your server's iDRAC IP addresses:

```
[servers]
192.168.1.100
```

6. Run the Ansible playbook:

Execute the Ansible playbook with the appropriate variables:

```bash
ansible-playbook -i inventory.ini update_firmware.yml --extra-vars "idrac_user=root idrac_password=your_idrac_password pxe_boot_server=your_pxe_boot_server_ip share_user=your_share_user share_password=your_share_password"
```

This playbook will update the firmware of the specified servers by connecting to the iDRAC, fetching the firmware files from the PXE boot server, and applying the updates. The servers will reboot as part of the process.

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
- name: Update Dell Server Firmware using Redfish
  hosts: servers
  gather_facts: no
  tasks:
    - name: Update firmware from a repository on a network share (HTTP)
      redfish_command:
        category: Update
        command: PushUpdate
        baseuri: "{{ inventory_hostname }}"
        username: "{{ redfish_user }}"
        password: "{{ redfish_password }}"
        update_uri: "http://{{ http_server }}/path/to/firmware/files/firmware_file_name"
        protocol: HTTP
```

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

This playbook will update the firmware of the specified servers by connecting to the iDRAC using Redfish API, fetching the firmware files from the HTTP server, and applying the updates. The servers might reboot as part of the process, depending on the firmware being updated.