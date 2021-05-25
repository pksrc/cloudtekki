---
title: "Hub Service Template"
date: 2021-05-23T00:00:00-00:00
draft: false
categories: [HUBSERVICES, EUC]
author: Nagul Subramanian
authorlink: /https://www.linkedin.com/in/nagulsubramanian 
tags:
- Access
- Hub Services
- Templates 
- Workspace ONE
lightgallery: true
---

## What is Hub Services? 
To simply state it helps enterprise introduce a true digital experience to their employees on any device or web. Hub Services currently offers the following set of features: 
- Unified catalog 
- Branding 
- Custom/ Home tab 
- Employee Self-service 
- Notifications 
- Passport (Require WS ONE Access configuration) 
- People (Require WS ONE Access configuration)
- Virtual Assistance (Require WS ONE Access configuration) and 
- New Hire Welcome (Require WS ONE Access configuration) 

This is just getting started with lot of potential to unlock to make the employee experience a world class. There are few docs that captures all the features in detail as referenced below, so with that I want to focus specifically on the Hub Services template in this post. 
- [Enabling and Delivering Hub Services with Workspace ONE Intelligent Hub](https://kb.vmware.com/s/article/2960170)
- [VMware Workspace ONE Intelligent Hub Series Part 1: Evolve from Legacy App Catalog as Web Clip to Hub Catalog](https://thomascheng.net/2020/07/17/evolve-from-legacy-app-catalog-as-web-clip-to-hub-catalog-within-vmware-workspace-one-intelligent-hub/)
## Why the need for Hub Services Templates?

In the initial release of the Hub Services, most of the features with exception to a few can be set at the Global level only and was not able to provide the customization per group or for a subset of devices to provide an user-centric experience and personalize/tailor content for each personas. So with introduction of Hub Services templates, we can provide that curated experience. As the name represents, it allows you to create templates with different set of features and/or configurations and can then be deployed to UEM smart groups or Access user groups or both. Let's take knowledge and remote worker personas as an example, each of their day-to-day functions differ and rely on unique resources to accomplish their task, such that it would be good to customize their Hub experience overall. Let's jump into the setup.. 

## Setup

Pre-requisite: If you haven't configured the Hub Services, please follow the instructions here: [Activating Hub Services](https://kb.vmware.com/s/article/2960170).

In this post, as previously referenced, I'm taking the knowledge and the remote worker personas as an example to create a template for each. The other common use case I see in the field is, admin creates template(s) for testing feature(s) with a small group of users prior to the Global rollout. 

### Step 1: Review Global Settings 
By default, there is a Global setting for each features. So if any of the features needs to be applied globally, you can go ahead and configure the Global setting accordingly. If it needs to be enabled only for targeted users or you would like to test with small group users prior to enabling it, then leverage templates and do not enable the Global setting. For example, if you would like to enable "People" for certain personas only, then leave the Global setting disabled. 

So back to our example, for both the knowledge and the remote workers, the branding would be the same. Such that I've adjusted the branding for the "Global" itself as shown in below screenshot. If you have subsidiary companies, you can create versions(as in step 2) and override the branding feature via templates. 
{{<image src="/img/euc/hub-templates/Global.PNG" caption="Global Settings">}}

### Step 2: Create New Versions 
For the following set of features,
- App Catalog 
- Branding
- Custom Tab and
- Employee Self-Service

you have the ability to create and maintain multiple versions which essentially used in templates to override the Global setting. NOTE that adding a new version should not modify the current assignments.  
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

### Step 4: Assign the Template 
The last step is to assign the template. As previously noted, you can assign the template to either smart groups or Access user group or both based on the setup you have and the experience you would like to provide
{{<image src="/img/euc/hub-templates/Assign_2.PNG" caption="Assignment">}}

{{< admonition type=note title="Note" open=true >}}
- UEM smart groups assignment - enables experience on iOS, Android, macOS and Windows platforms 
- Access user group assignment - enables Web experience (require Access integration and Access to set as Auth) 
{{< /admonition >}}

Note that the priority determines which template is applied when users or devices are assigned to multiple templates.

### Result:
- As a Knowledge Worker, I see the Home tab to be personalized along with People tab enabled 
{{<image src="/img/euc/hub-templates/KW_2.PNG" caption="Knowledge Worker Experience">}}
- As a Remote Worker, I see the Home tab to be personalized but no access to People tab
{{<image src="/img/euc/hub-templates/Sales.PNG" caption="Remote Worker Experience">}}

{{< admonition type=note title="Note" open=true >}}
- For Hub and UEM requirements, please refer [Hub Releases Notes](https://docs.vmware.com/en/VMware-Workspace-ONE/services/rn/VMware-Workspace-ONE-Hub-Services-Release-Notes.html#about). In general, iOS fully supports templates today however Android do not but soon we will be. Will update the post once the specific versions are confirmed.   
{{< /admonition >}}

## Reference
- [Hub Services Documentation](https://docs.vmware.com/en/VMware-Workspace-ONE/services/intelligent-hub_IDM.pdf)
- [Hub Services Templates Feature walk-through video](https://techzone.vmware.com/?share=video3022)
## Future blog posts: 
- Experience Workflow
- New Hire Onboarding 

