---
title: "Hub Service Template"
date: 2020-10-24T00:00:00-00:00
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
To simply state it helps enterprise introduce a true digital experience to their employees on any device or web. This is just getting started with lot of potential to unlock features to make their employee experience a world class. 

There are few docs that captures all the features in Hub Services in detail as referenced below, so with that I want to focus specifically on the Hub Services template in this post. 
- [Enabling and Delivering Hub Services with Workspace ONE Intelligent Hub](https://kb.vmware.com/s/article/2960170)
- [VMware Workspace ONE Intelligent Hub Series Part 1: Evolve from Legacy App Catalog as Web Clip to Hub Catalog](https://thomascheng.net/2020/07/17/evolve-from-legacy-app-catalog-as-web-clip-to-hub-catalog-within-vmware-workspace-one-intelligent-hub/)
## Why we need the Hub Services Templates?

- To truly provide an user-centric experience we need a way to personalize/tailor content for each personas. Let's take the knowledge worker and the sales personas as an example, each of their day-to-day functions differ and rely on unique resources to accomplish their task. Such that it is important to provide a curated experience and that can achieved via the Hub Services template. As the name represents, it allows you to create templates with different set of features and/or configurations and can then be deployed to UEM smart groups or Access user groups or both 

{{< admonition type=note title="Note" open=true >}}
- UEM smart groups assignment - applicable for iOS, Android, macOS and Windows platform 
- Access user group assignment - applicable for Web experience (require Access integration and Access to set as Auth) 
{{< /admonition >}}

## Setup

If you haven't configured the Hub Services, please follow the instructions here: [Activating Hub Services](https://kb.vmware.com/s/article/2960170).

In this post, as previously referenced, I'm taking the knowledge worker and the sales personas as an example to create a template for each. The common use case I see in the field is, admin creates template(s) for testing feature(s) with a small group of users prior to the Global rollout. 

### Step 1: Review Global Settings 
By default, there is a Global setting for each features. If any of the features needs to be applied globally, you can go ahead and configure the Global setting accordingly. If it needs to be enabled only for targeted users, leverage templates and do not enable the Global setting. For example, if you would like to enable "People" for certain personas only, then leave the Global setting disabled. 

So back to our example, for both the knowledge worker and the sales, the branding would be the same. Such that I've adjusted the branding for the "Global" itself as shown in below screenshot. If you have subsidiary companies, you can create versions( as in step 2) and override the branding feature in the templates. 
{{<image src="/img/euc/hub-templates/Global.PNG" caption="Global Settings">}}

### Step 2: Create New Versions 
For the following set of features,
- App Catalog 
- Branding
- Custom Tab and
- Employee Self-Service
you have the ability to create and maintain multiple versions which essentially used in templates to override the Global setting. NOTE that adding a new version do not modify the current assignments.  
For our use case, I'm creating new versions of App Catalog and Custom Tab for the Knowledge worker and the sales rep personas. Click "Add Version" and adjust the configurations as needed. 
For Knowledge Worker - 
- I'm creating a new version of App Catalog prioritizing Promotions 
{{<image src="/img/euc/hub-templates/AppCatalog_KW.PNG" caption="AppCatalog Knowledge Worker">}}
- and for Home tab, pointed the URL to Knowledge Worker home page 
{{<image src="/img/euc/hub-templates/home_KW.PNG" caption="Home Tab Knowledge Worker">}}

For Sales, 
- I'm prioritizing New Apps, Recommended and Business for the App Catalog  
{{<image src="/img/euc/hub-templates/AppCatalog_Sales.PNG" caption="AppCatalog Sales">}}
- and for Home tab, pointed the URL to Sales home page
{{<image src="/img/euc/hub-templates/home_Sales.PNG" caption="Home Tab Sales">}}

### Step 3: Create a Template
In this step, we will be creating templates for each personas with the feature set(s) that needs to override the Global settings as follows: 

- Knowledge Worker - here I've set Branding to Global, App Catalog to respective version, Home to respective version and enabled People 
{{<image src="/img/euc/hub-templates/template_KW_2.PNG" caption="Template Knowledge Worker">}}
- Sales - here I've set Branding to Global, App Catalog to respective version, and Home to respective version
{{<image src="/img/euc/hub-templates/template_Sales_2.PNG" caption="Template Sales">}}

### Step 4: Assign the Template 
The last step is to assign the template. As previously noted, you can assign the template to either smart groups or Access user group or both based on the setup you have and the experience you would like to provide
{{<image src="/img/euc/hub-templates/Assign.PNG" caption="Assignment">}}

Note that the priority determines which template is applied when users or devices are assigned to multiple templates.

Result:
- Knowledge Worker 
{{<image src="/img/euc/hub-templates/KW.PNG" caption="Knowledge Worker Experience">}}
- Sales 
{{<image src="/img/euc/hub-templates/Sales.PNG" caption="Sales Experience">}}

{{< admonition type=note title="Note" open=true >}}
- iOS fully supports Hub Templates 
- Android require Hub 21.03+ and UEM 21.05+ 
{{< /admonition >}}

## Reference
- [Hub Services Documentation](https://docs.vmware.com/en/VMware-Workspace-ONE/services/intelligent-hub_IDM.pdf)
- [Hub Services Templates Feature walk-through video](https://techzone.vmware.com/?share=video3022)
## Future blog posts: 
- Experience Workflow
- New Hire Onboarding 

