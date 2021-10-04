---
title: "Registered mode w/ a focus on Windows 10"
date: 2021-10-03T00:00:00-00:00
draft: false
categories: [EUC]
tags:
- Windows10
- RegisteredMode
lightgallery: true
featured: true
featuredImage: /img/euc/registered-mode/hero.png
---

<p class="cap">
Registered mode is a capability in UEM that allows IT to deliver a secure way for Employees to access corporate data using Workspace ONE without having to manage their devices. While the core philosophy behind Registered mode is the same across different platforms, the problem it addresses on various platforms are slightly different. Understanding its origin will shed some light into how you can effectively utilize this capability and provide seamless access to your Employees
</p>

## Evolution of Registered Mode

As a technologist that's been following and absorbing all the progress that's been happening this space, I wanted to start this post by covering the evolution of Registered Mode with Workspace ONE over the last 4-5 years. 

### iOS
Registered Mode was first introduced on iOS. If you have enrolled an iOS device, a consumer privacy focussed platform, you know that you feel like you are signing your life away when you go through the Device Enrollment process. Apple's enrollment language created a lot of privacy concerns amongst the users and in one instance, I've witness a technically savvy UEM Administrator refusing to enroll his own BYO device while understanding fully well that no personal information is collected on their particular tenants. Data that can be collected by UEM is provided [here](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2011/WS1EX/GUID-AWT-AWEX-PRIVACY.html?hWord=N4IghgNiBcIAoCcCWA3MBjAniAvkA)

VMware looked to address this through a few different ways
1. [Privacy Notice](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2011/UEM_Managing_Devices/GUID-AWT-PRIVACYNOTICE.html) - This aims to inform Users  about the data that is being collected by UEM (this could vary from one device to another for the same Employee based on device ownership) 
2. [Airwatch Container](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2008/iOS_Platform/GUID-AWT-APPSWSFORIOS.html) ([now unavailable for download](https://kb.vmware.com/s/article/84089))- The original MAM only container aimed to provide Mobile Application Management (MAM) only capability. A layer of Application based security was available through the SDK. 
3. Workspace ONE App ([now EOSL](https://kb.vmware.com/s/article/80208))- This was the next evolution to the AirWatch Container and intended to provided a MAM only layer of management which meant that Users could get access to Corporate Resources in a limited way. Additionally, this helps you deliver a Unified App Catalog - Native Apps from UEM, Federated Web Apps from Access and Virtual Apps from Horizon/Citrix

The best approach yet is consolidating all the experience through the Intelligent Hub App to build the **Registered Mode on iOS**. If configured by the Admin, an Employee would authenticate to the Intelligent Hub App to see what they are entitled to get and have access to. Employees could step up their device to Management if your Corporate deemed it necessary for subset of Apps. 

### Android

On Android, the story becomes slightly different... 

The Android ecosystem has normalized around the Android Enterprise solution set that was introduced by Google in an attempt to unify the management story on a segmented Android landscape. When it comes to delivering a BYO experience on Android devices you could do so using the Work Profile or COPE within the Android Enterprise framework. The **BYO experience** however is **NOT an option in China** due to the Great Firewall of China which blocks Google's endpoints which are required to create the appropriate managed google accounts to deliver the Android Enterprise capabilities there. 

> NOTE: For Corporate Owned devices, there is an option to deliver Android Experience using the [Work Managed Device Without Google Play Services](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2011/Android_Platform/GUID-AWT-AFWDEVICEMODES.html#work-managed-device-without-google-play-services-2) option 

This limitation and the need to deliver access to corporate data on Mobile to Users in China brought about the Registered Mode on Android. With Registered Mode, IT can provide a way for Users to get access to Email (through Boxer), File Access (through Content Locker), Secure Browsing of Internal resources (through Web) and all the capabilities shipped with the WS1 Intelligent Hub with Hub Services. 

A couple of nuances to stay aware of
- Given the lack of access to the Google Play Store in China, Users can get their Workspace ONE Apps from [getwsone.com](getwsone.com) (more details [here](https://kb.vmware.com/s/article/75197))
- If you are using Access as for AuthN for Mobile devices or All devices for Users, there isn't a way to provide a conditional access policy that can be applied for Registered devices selectively. 


### Windows

The next domino to fall is obviously Windows for very different reasons. If you think about it, there are not a lot of BYO use cases around a Windows device. Most Employees today that need a Windows PC gets a laptop which is going to be the primary device to get work done. What then drove the need for an Registered mode on Windows? The [answer](https://blogs.vmware.com/euc/2019/11/workspace-one-microsoft-endpoint-manager.html) remains the same but the scope changes slightly - "Deliver WS1 capabilities on a **Corporate Owned device** (not BYO devices) where the device might be managed by another UEM software". 

> NOTE: Registered Mode on Windows is not available for BYO devices (some capabilities may work but not supported)

## Registered Mode on Windows

With that in mind, let's explore the capabilities that Workspace ONE Registered mode on Windows 10 provide.. Outside of a brief mention to the capabilities in [VMware docs](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/2011/Windows_Desktop_Device_Management/GUID-036EFAD5-03F5-44D1-926E-F3F45754B9F5.html) information about Registered mode for Workspace ONE is limited. 

### What is not available? 

This is easy so let's get this out of the way first - There are no native MDM capability through Workspace ONE with the registered mode - no apps, no profiles, no compliance.

### What is available?

Here are the capabilities of Registered Mode on Windows that have been validated to be working

 - Enrollment into **Registered Mode** 
 - Secure Access to corporate resources within Internal Network through **Workspace ONE Tunnel**
   - NOTE: Tunnel VPN profile needs to be configured and this Profile installation worked on a Registered Mode device (Solution is architected to deliver certificate without OMADM)
   - NOTE: Workspace ONE Tunnel app will be deployed using the Management software on the device
 - Deliver Remote Assistance for troubleshooting using **Workspace ONE Assist**
   - NOTE: Workspace ONE Assist app will be deployed using the Management software on the device
 - Digital Employee Experience Management (**DEEM**)
   - NOTE: Device Ownership needs to be set to Corporate Owned
 - Employee Experience through **Hub Services**
 - Ability to gather data about the device using **Sensors**
   - NOTE: Sensors in System context will work and those in User context doesn’t work. Also, Device Ownership needs to be set to Corp Owned
 - Ability to configure devices using **Baselines**

## Enabling Registered Mode in WS1 UEM

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/admin.png" caption="Setting up Registered Mode for Windows">}}
{{<image src="/img/euc/registered-mode/admin_1.png" caption="Setting up the Tunnel Profile for Windows">}}

## End User Enrollment into Registered Mode

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/enroll_5.png" caption="Enter Email or Server Address">}}
{{<image src="/img/euc/registered-mode/enroll_4.png" caption="Enter Username and Password">}}
{{<image src="/img/euc/registered-mode/enroll_3.png" caption="Choose Ownership - Remember this must be set to Corporate Owned for DEEM and Sensors">}}
{{<image src="/img/euc/registered-mode/enroll_2.png" caption="Data Collection Prompt">}}
{{<image src="/img/euc/registered-mode/enroll_1.png" caption="Successfully Enrolled!">}}

## Management Option in UEM

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/manage_3.png" caption="Management Options for Administrator">}}
{{<image src="/img/euc/registered-mode/manage_2.png" caption="A look at Baselines and More Options">}}
{{<image src="/img/euc/registered-mode/manage_1.png" caption="Management of Profiles (Tunnel Only)">}}
{{<image src="/img/euc/registered-mode/manage.png" caption="Options for Remote Management">}}

## End User Experience

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/enduser_5.png" caption="Hub Experience - Favorites">}}
{{<image src="/img/euc/registered-mode/enduser_4.png" caption="Hub Experience - People Search">}}
{{<image src="/img/euc/registered-mode/enduser_3.png" caption="Hub Experience - Support tab">}}
{{<image src="/img/euc/registered-mode/enduser_1.png" caption="Tunnel Connected on Device as configured for DTR">}}

## Remote Management Admin Experience

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/rm_admin_5.png" caption="Remote Management Options for Administrator">}}
{{<image src="/img/euc/registered-mode/rm_admin_4.png" caption="Initiating a Remote Screen Share">}}
{{<image src="/img/euc/registered-mode/rm_admin_3.png" caption="Connecting to a Device">}}
{{<image src="/img/euc/registered-mode/rm_admin_2.png" caption="Attend Mode PIN for Remote Screen Share">}}
{{<image src="/img/euc/registered-mode/rm_admin_1.png" caption="Remote Management Screen Share Admin Experience">}}

## Remote Management End User Experience

>*Explained through screenshots. Click through the pictures for a better experience*

{{<image src="/img/euc/registered-mode/rm_enduser_2.png" caption="Remote Management PIN prompt">}}
{{<image src="/img/euc/registered-mode/rm_enduser_1.png" caption="Remote Management Screen Share User Experience">}}