---
title: "Automating Windows 10 VM build auto enrolled into UEM using Packer"
date: 2021-02-22T00:00:00-00:00
draft: false
categories: [EUC]
tags:
- Automation
- Windows
- Packer
- UEM
featured: false
---

In this post, I'll be covering how you can automate the process of building a Windows 10 images on your Lab. As a part of my daily job working with Workspace ONE Unified Endpoint Management, I work with a lot of Modern Management use cases and this requires testing various use cases on various versions of Windows 10. To always start with a clean image, I use Packer to build and enroll the device for me into my lab UEM tenant thus speeding up the process and reducing the probability for errors. 

Let's dive into how you can get started with Packer to automate the entire Windows 10 build process along with enrolling into Workspace ONE UEM without a single click...

## What is Packer? 
If you are not familiar with Packer, [Packer](https://www.packer.io/) by Hashicorp is a tool that helps automate the images building process for multiple platforms such as AWS, Azure, GCP, VMware(all at the same time, if you choose to). With Packer you can attempt to build a very consistent image every time by automating the steps to build an image through code. 

### Packer Use Cases
Several use cases are highlighted [here](https://www.packer.io/intro/use-cases)
- [Continuous Delivery](https://www.packer.io/intro/use-cases#continuous-delivery)
- [Dev/Prod Parity](https://www.packer.io/intro/use-cases#dev-prod-parity)
- [Appliance/Demo Creation](https://www.packer.io/intro/use-cases#appliance-demo-creation)

## How do I use it? 
1. Since I have the need to work with a lot of Windows 10 (in EUC) for Endpoint Management, I use the Packer to build a VM for the version of Windows 10 that I need in under 30 minutes. My homelab has little storage and thus rather than creating templates, I leaned heavily on Packer which helps minimizing the storage used. Once I have the working set of build files for an Operating System + version, I can always replicate the Image (most of the time barring some unexpected OS update behaviors).  
2. Packer is also used to produce the [VMware Event Broker Appliance](vmweventbroker.io) and it was in fact William Lam {{% twitter profile="lamw" %}} who first introduced me to the magic of Packer and how capable it is to build consistent images

## Getting started
You can get started by installing packer on your operating system as described [here](https://learn.hashicorp.com/tutorials/packer/getting-started-install)

### macOS Installation
```sh
brew tap hashicorp/tap
brew install hashicorp/tap/packer
```

## Terminologies

If you are going to work with Packer, you'll need to know these common terminologies

- **Variables**: self explanatory - define any and all your variables here.. these will be used in the following sections
- **Builder(s)**: These are platform specific using various APIs to product consistent images across various platforms such as AWS, Azure, GCP, VMware
- **Provisioners**: does stuff to configure the image after booting - such as installing packages
- **Post Processors**: runs after the image is built - such as exporting the VM as an OVA

## Template for Windows

Let's go over the details of the Packer template that I've leveraged for the Windows 10 build. You can find the template that I've used in my github repo - https://github.com/pksrc/packer 

```
git clone https://github.com/pksrc/packer.git 
cd packer/win10
```

### Variables
All the variables that we are going to use are split up into two files for readability
- `var-appliance.json` - has all the information about the Appliance such as name, CPU, disk size etc

```json
{
    "version": "0.1.0",
    "description": "Windows Image",
    "vm_name": "Windows_10",
    "numvcpus": "2",
    "memory": "4096",
    "disk_size": "50720",
    "guest_username": "vmuser",
    "guest_password": "VM@Windows10",
    "autounattend": "./answer/10/Autounattend.xml"
    ....
}
```

- `var-builder.json` - has all the information about the platform where we are going to build the VM

```json
{
    "vcenter_server": "10.10.X.X",
    "vcenter_username": "administrator@vsphere.local",
    "vcenter_password": "P@ssword",
    "VM_folder": "Templates",
    "esx_host": "CMD/esxi03.lab.pdotk.com",
    "resource_pool": "",
    "datastore": "datastore2-30",
    "network": "dPG-Access" #Connect to the Network with DHCP enabled
}
```

### vSphere ISO Builder
We are going to be using the vSphere 7.0 platform for building our VMs and thus using `vsphere-iso` which leverages vSphere API (supports vSphere 6.5+).

> If you don't have a vCenter and are going to be building directly on your ESXi host(s) you can leverage the `vmware-iso` which requires some ESXi modification as it needs SSH access and leverages VNC

Let's now walk through all the options specified below 
- **type**: indicates the builder that we are going to leverage and we are going to be using `vsphere-iso` (obviously)
- **build platform information** - information about which vCenter we are going to connect to, the necessary credentials, what datastore and resource pool are going to be used, is vCenter certificate trusted etc.
```json
{
    "type": "vsphere-iso",
    "vcenter_server": "{{user `vcenter_server`}}",
    "host": "{{user `esx_host`}}",
    "resource_pool": "{{user `resource_pool`}}",
    "datastore": "{{user `datastore`}}",
    "username": "{{user `vcenter_username`}}",
    "password": "{{user `vcenter_password`}}",
    "insecure_connection": "true", 
    ....
}
```

- **ISO**: You'll need to provide two ISOs for the Windows Image to build succesfully
  1. **The bootable Windows 10 ISO** - you can provide both the local downloaded file path and/or the remote URL to download from. A checksum to the file needs to be provided for added security. 
  2. Since **VMware Tools** will also be needed, we'll need to upload the VMware Tools to the datastore and mount it using `iso_paths` attribute. A word of caution as there are no validations for the presence of files on the datastore. If you don't have the file you could be staring at a black screen on the VM for a long time wondering why the boot screen is not coming up, ask me how I know.. 

```json
{
    ....
    "iso_urls": [
        "{{user `local_iso_path`}}",
        "{{user `remote_iso_url`}}",
    ],
    "iso_checksum": "db756063e9efa39c68501f745c2ea9c0",
    "iso_paths": [
        "[{{user `datastore`}}] {{user `vmtools_windows`}}"
    ],
    ...or...
    "iso_paths": [
        "[{{user `datastore`}}] {{user `iso_url`}}",
        "[{{user `datastore`}}] {{user `vmtools_windows`}}"
    ],
    ....
}
```

- **VM information** - Name, VM version, Guest OS Type, CPU, RAM, storage and networking information

```json
{
    ....
    "vm_name": "{{user `vm_name`}}",
    "notes": "{{user `description`}} - {{user `version`}}",
    "vm_version": 14,
    "guest_os_type": "windows9_64Guest",
    "cpus": "{{user `numvcpus`}}",
    "ram": "{{user `memory`}}",
    "disk_controller_type": "pvscsi",
    "storage": [
        {
            "disk_size": "{{user `disk_size`}}",
            "disk_thin_provisioned": true
        }
    ],
    "network_adapters": [
        {
            "network": "{{user `network`}}"
        }
    ],
    ....
}
```

- **Seed Files**: We can seed some files through the floppy files and these are then leveraged during the `autounattend` setup

```json
{
    ....
    "floppy_files": [
        "{{user `autounattend`}}",
        "./floppy/WindowsPowershell.lnk",
        "./floppy/PinTo10.exe",
        "./floppy/vmtools.cmd",
        "./scripts/fixnetwork.ps1",
        "./scripts/disable-screensaver.ps1",
        "./scripts/disable-winrm.ps1",
        "./scripts/enable-winrm.ps1",
        "./scripts/microsoft-updates.bat",
        "./scripts/win-updates.ps1"
      ],
      "floppy_img_path": "[{{user `datastore`}}] VMware-Tools-windows-11.0.0-14549434/floppies/pvscsi-Windows8.flp",
    ....
}
```

- **WinRM config** - options configuring the WinRM communication that we need for connecting the the VM after booting - how long should Packer wait for a reboot, how should packer authenticate with the Guest etc.

```json
{
    ....
    "communicator": "winrm",
    "winrm_timeout": "{{user `winrm_timeout`}}",
    "winrm_password": "{{user `guest_password`}}",
    "winrm_username": "{{user `guest_username`}}",
    "boot_wait": "{{user `restart_timeout`}}",
    "shutdown_command": "shutdown /s /t 10 /f /d p:4:1 /c \"Packer Shutdown\"",
    "ip_wait_timeout": "{{user `ip_wait_timeout`}}",
    ....
}
```

### AutoUnattend File

Every OS has its own method to support an automated build. For the Windows, we want to focus on the `autounattend` file as described [here](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/automate-windows-setup) and [here](https://www.packer.io/guides/automatic-operating-system-installs/autounattend_windows) 

It might be hard to cover the `Unattend` file at length and thus I've provided the high-level details along with how there are utilized below (excerpt from MSFT [documentation](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs) where you can easily get lost in the labyrinth of hyperlinks)

- **windowsPE**: These settings are used by the Windows Setup installation program. We use this to [install the PVCSCI drive](https://kb.vmware.com/s/article/1010398) which is our `disk_controller_type`

- **specialize**: You can have multiple blocks of `specialize` 
  - These settings are triggered both at the beginning of audit mode and at the beginning of OOBE. We use this to setup the disk partitions, language settings, User settings, Product Key etc. You could also setup static IP (which I'm not using)
  - Use to install VMware Tools on the Guest OS
  - Use for Naming the VM

- **oobeSystem**: The suggestion is to use this sparingly however we use this heavily to configure the system after the user completes OOBE. 

A good reference to an example Autounattend file and all the helper scripts are available in Stefan Scherer's repo - https://github.com/StefanScherer/packer-windows (forked from https://github.com/joefitzgerald/packer-windows). This repo provides examples that uses the `vmware-iso` builder that is being utilized. My repo has a modified version of this to help build succesfully on vSphere - https://github.com/pksrc/packer 

```
git clone https://github.com/pksrc/packer.git 
cd packer/win10
```

## Starting the Build

Starting the build is easily done with the `packer` command which should kick off the build process on vCenter using the configuration provided

```
packer build \
    --only=vsphere-iso \ #if there are multiple builders, this builds only the vsphere-iso configuration
    --var-file=var-builder.json \ 
    --var-file=var-appliance.json \ 
    --var iso_url=packer_cache/en_windows_10_business_editions_version_2004_x64_dvd_d06ef8c5.iso \ 
    --var vm_name="Windows10_2004" \ 
    windows_10.json
```

### Build process 
As a part of the build process, 
- the Windows 10 ISO is uploaded to vCenter onto the datastore specified 
- the VM is created and booted up from the ISO uploaded
- the VMware Tools is first installed and forces a reboot
- the Windows Setup begins leveraging the autounattend and the supporting script files made available in the floppy drives
- the autounattend process also runs the scripts to help configure some settings on the VM
- the VM is configured to auto logon with a User specified in the AutoUnattend.xml
- Packer is able to acquire the IP once the Windows is configured on the VM
- the provisioning stage now kick in and starts to execute each block - it is here that we attempt to also enroll the VM into UEM
- once the provisioning stage is over, the post processor kicks in - I'm only writing a success message here

The entire process took 30 minutes can be seen in this video(sped up to 5 mins) below -  
{{% youtube pVUVSoi1J2U %}}

### Enrollment into UEM
Using the Post provisioning step, we tell packer to run the below script which will download the `AirwatchAgent.msi` and enroll the device into Workspace ONE UEM. 

```bat
if not exist "C:\tmp\AirWatchAgent.msi" (
	powershell -Command "(New-Object System.Net.WebClient).DownloadFile('https://packages.vmware.com/wsone/AirwatchAgent.msi', 'C:\tmp\AirWatchAgent.msi')" <NUL
)

rem Check if device is already registered with Workspace ONE, if not then proceed with installing Workspace ONE Intelligent Hub

for /f "delims=" %%i in ("reg query HKLM\SOFTWARE\Microsoft\Provisioning\OMADM\Accounts /s") do set status=%%i

if not defined status goto INSTALL

:INSTALL
    rem Run the Workspace ONE Intelligent Hub Installer to Register Device with Staging Account

    msiexec /i C:\tmp\AirWatchAgent.msi /q ENROLL=Y SERVER=uem.lab.pdotk.com LGName=customerog USERNAME=test@lab.pdotk.com PASSWORD=P@ssw0rd DOWNLOADWSBUNDLE=TRUE /LOG %temp%\WorkspaceONE.log
```

## Wrap Up

Hopefully this post helps you get started with packer quickly and with your Windows 10 use cases. I've shown an example where I was able to auto enroll into UEM and have a fresh image always ready to test within a span of 30 minutes but the possibilities with automation enabled by packer are endless... Hoping you'll give it a try! :smile:

### PS:Upgrading Packer to 1.7.0
I've been using Packer for 6-9 months now and had finalized my templates mostly. However at the time of posting this, Packer has introduced some changes with v1.7.0 which suggests an template upgrade. I'll be covering these changes in the next blog post. 