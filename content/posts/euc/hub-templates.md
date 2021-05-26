---
title: "An Intro to Workspace ONE Hub Templates"
date: 2021-05-25T00:00:00-00:00
draft: false
categories: [EUC]
author: Nagul Subramanian
authorlink: https://www.linkedin.com/in/nagulsubramanian 
tags:
- Access
- Hub Services
- Templates 
- Workspace ONE
lightgallery: true
---

Hi! I'm Nagul Subramanian working on VMware End User Computing technologies for more than 7 years and have gained a lot of field experience on WS ONE and Mobility. I've always had a personal goal to contribute my knowledge back to the community. As I explored ways to do that, I ran into PK and Cloud Tekki. This is my first blog post(hopefully first of many) and would love to hear your feedback!  

## What is WS ONE Hub Services? 
Workspace ONE Hub Services helps enterprise introduce a true digital experience to their employees on any device or web. Hub Services currently offers the following set of features: 
- Unified catalog 
- Branding 
- Custom/ Home tab 
- Employee Self-service 
- Notifications 
- Passport (Require WS ONE Access configuration) 
- People (Require WS ONE Access configuration)
- Virtual Assistance (Require WS ONE Access configuration) and 
- New Hire Welcome (Require WS ONE Access configuration) 

This is just getting started with lot of potential to unlock value to create a world class employee experience. There are few docs that captures all the features in detail as referenced below, so with that I want to focus specifically on the Hub Templates in this post. 
- [Enabling and Delivering Hub Services with Workspace ONE Intelligent Hub](https://kb.vmware.com/s/article/2960170)
- [Hub Services Feature walk-through video](https://techzone.vmware.com/?share=video3005)
- [VMware Workspace ONE Intelligent Hub Series Part 1: Evolve from Legacy App Catalog as Web Clip to Hub Catalog](https://thomascheng.net/2020/07/17/evolve-from-legacy-app-catalog-as-web-clip-to-hub-catalog-within-vmware-workspace-one-intelligent-hub/)

## Why Hub Templates?

In the initial release of the Hub Services, most of the features with some exceptions can only be set at the Global level. Customers were not able to provide the customization per group or for a subset of devices to provide an user-centric experience and personalize/tailor content for each personas. So with the introduction of Hub templates, we can deliver that curated experience. As the name suggests, it allows you to create templates with different set of features and/or configurations to then be deployed to UEM smart groups or Access user groups or both. 

With the latest version of Intelligent Hub and Hub services, Hub Templates is available on all platforms - iOS, Android, Mac, and Windows devices and on a Web browser. For more specific guidance around versions of Intelligent Hub and Hub services supporting Hub Templates, please refer [Hub Releases Notes](https://docs.vmware.com/en/VMware-Workspace-ONE/services/rn/VMware-Workspace-ONE-Hub-Services-Release-Notes.html#about).

Let's jump into the setup.. 

## Setup

**Pre-requisite**: If you haven't configured the Hub Services, please follow the instructions here: [Activating Hub Services](https://kb.vmware.com/s/article/2960170).

Let's take Knowledge and Remote Worker personas as an example, each of their day-to-day functions differ and rely on unique resources to accomplish their task. We are going to explore how to leverage templates to customize their overall Hub experience. The other common use case I see in the field is creating template(s) for testing feature(s) with a small group of users prior to the larger rollout. 

### Step 1: Review Global Settings 
By default, there is a Global setting for each feature. So if any feature needs to be applied Globally, you can configure the Global setting accordingly. If it needs to be enabled only for targeted users or to small group of test users, then leverage Hub Templates and do not enable the Global setting. For example, if you would like to enable "People" for certain personas only, then leave the Global setting disabled. 

So back to our example, for both the Knowledge and the Remote Workers, the branding needs to the same. To achieve that I've adjusted the branding for the "Global" setting as shown in below screenshot. If you have subsidiary companies, you can create multiple versions (as in step 2) and override the branding feature via templates. 
{{<image src="/img/euc/hub-templates/Global.PNG" caption="Global Settings">}}

### Step 2: Create New Versions 
For the following set of features, you have the ability to create and maintain multiple versions which essentially used in templates to override the Global setting. NOTE that adding a new version should not modify the current assignments
- App Catalog 
- Branding
- Custom Tab and
- Employee Self-Service

For our use case, I'm creating new versions of App Catalog and Custom Tab. 

For Knowledge Worker:
- I'm customizing the App Catalog to include Promotions section 
{{<image src="/img/euc/hub-templates/AppCatalog_KW.PNG" caption="AppCatalog Knowledge Worker">}}
- and for Home tab, pointed the URL to specific landing page  
{{<image src="/img/euc/hub-templates/home_KW.PNG" caption="Home Tab Knowledge Worker">}}

For Remote Worker: 
- I'm adding a specific category(Business) to the App Catalog section  
{{<image src="/img/euc/hub-templates/AppCatalog_RW.PNG" caption="AppCatalog Remote Worker">}}
- and for the Home tab, similar to Knowledge Worker, pointed the URL to a different landing page
{{<image src="/img/euc/hub-templates/home_RW.PNG" caption="Home Tab Remote Worker">}}

### Step 3: Create a Template
In this step, we will be creating templates for each personas with the feature set(s) that needs to override the Global settings as follows: 

- Knowledge Worker - here I've set Branding to Global, App Catalog to respective version, Home to respective version and enabled People 
{{<image src="/img/euc/hub-templates/template_KW_2.PNG" caption="Template Knowledge Worker">}}
- Remote Worker - here I've set Branding to Global, App Catalog to respective version, and Home to respective version
{{<image src="/img/euc/hub-templates/template_RW_2.PNG" caption="Template Remote Worker">}}

### Step 4: Assign the Template gs
The last step is to assign the template. As previously noted, you can assign the template to either smart groups or Access user group or both based on the setup you have and the experience you would like to provide
{{<image src="/img/euc/hub-templates/Assign_2.PNG" caption="Assignment">}}

{{< admonition type=note title="Things to watch out for with assignments - from VMware docs" open=true >}}
- When the authentication mode is set to Workspace ONE UEM and Workspace ONE Access are not integrated, you can assign UEM smart groups. The Intelligent Hub experience on web browsers is not available.
- When the authentication mode is set to Workspace ONE Access and Workspace ONE Access and Workspace ONE UEM are integrated, you can assign UEM smart groups and Workspace ONE Access user groups. If you assign smart groups to a template, the template
definition is available in the Intelligent Hub app for iOS, Android, Mac, and Windows devices. Assign Workspace ONE Access user groups to the template to provide the same experience
on web browsers. Assigning user groups with smart groups to a template provides a consistent experience for users across iOS, Android, Mac, and Windows devices and on a web browser.
- When Workspace ONE Access is not integrated with Workspace ONE UEM, you can assign user groups. In this case, the Intelligent Hub experience on iOS, Android, Mac, and Windows Intelligent Hub apps is not available. Only the web browser experience is available.

Source: [How to Add a Hub Template and Assign It to Groups](https://docs.vmware.com/en/VMware-Workspace-ONE/services/intelligent-hub_IDM/GUID-58C04CC7-84DA-42C0-AD7C-9FFCC2026378.html)

{{< /admonition >}}
Additionally, the priority determines which template will be applied when users or devices are assigned to multiple templates.
### Result
Now let's see how the configurations translates into end users experience. 
- As a Knowledge Worker, I see the Custom Home tab along with People tab enabled 
{{<image src="/img/euc/hub-templates/KW_2.PNG" caption="Knowledge Worker Experience" width="400px">}}
- As a Remote Worker, I see the Custom Home tab but no access to People tab
{{<image src="/img/euc/hub-templates/Sales.PNG" caption="Remote Worker Experience" width="400px">}}

## References
- [Hub Services Documentation](https://docs.vmware.com/en/VMware-Workspace-ONE/services/intelligent-hub_IDM.pdf)
- [Hub Templates Feature walk-through video](https://techzone.vmware.com/?share=video3022)

I hope to be spending some more time on People Search, Experience Workflows, and New Hire Onboarding in the near future and will be documenting them in future blog posts 
