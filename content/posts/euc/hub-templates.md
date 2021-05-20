---
title: "Hub Service Template"
date: 2020-10-24T00:00:00-00:00
draft: false
categories: [HUBSERVICES, EUC]
author: Guest
authorlink: /https://www.linkedin.com/in/nagulsubramanian 
tags:
- Access
- Hub Services
- Templates 
- Workspace ONE
lightgallery: true
---

## What is Hub Services? 
To simply state it helps enterprise introduce a true digital experience to their workforce on any device or web. This is just getting started with lot of potential to unlock features that truly enables enterprise to make their employee experience a world class. 

There are few blogs that captures Hub Services in detail as referenced below so with that I want to focus on the Hub Services template in this post. 

## Why Hub Services Templates?

- To truly provide an user-centric experience we need a way to personalize/tailor content for each personas in the workforce. For example, the sales rep vs the HR, their day-to-day functions differ and each rely on unique resources to accomplish their task. Such that it is important to provide a curated experience and that can delivered via Hub Services template 
- As the name represents, it allows you to create templates with different set of features and/or configurations. It can then be deployed to UEM smart groups or Access user groups or both 

{{< admonition type=note title="Note" open=true >}}
- UEM smart groups - applicable for experience in iOS, Android, macOS and Windows platform 
- Access user group - applicable for Web experience (require Access integration and Access to set as Auth) 
{{< /admonition >}}

## Setup

If you have setup Hub Services, please skip the following step. If not, follow the instructions here:  

I've taken couple of common personas, the knowledge worker and the sales, as an example to create a template for each in this post. The other common use case is, admins create template for testing. 

### Step 1: Review Global Settings 
For each features, there will be a Global setting with default configurations as shown below. Review and make the adjustments as necessary.
{{<image src="/img/euc/hub-templates/Global.PNG" caption="Global Settings">}}

In my case, for both the knowledge worker and the sales, the branding would be the same. Such that I've adjusted the branding for the "Global" itself. 

### Step 2: Create New Versions 
For the following set of features,
- App Catalog 
- Branding
- Custom Tab and
- Employee Self-Service
you have the ability to create and maintain multiple versions per persona requirements which essentially can be used to override the Global setting. NOTE that adding a new version do not modify the current assignments.  
For our use case, I'm creating new versions of App Catalog and Home tab for the Knowledge worker and the sales rep personas. Click "Add Version" and adjust the configurations as shown below. 

For Knowledge Worker - 
- I'm creating a new version of App Catalog and prioritizing Promotions. 
{{<image src="/img/euc/hub-templates/AppCatalog_KW.PNG" caption="AppCatalog Knowledge Worker">}}
- and for Home tab, pointed the URL to Knowledge Worker home page 
{{<image src="/img/euc/hub-templates/home_KW.PNG" caption="Home Tab Knowledge Worker">}}

For Sales, 
- I'm creating a new version of App Catalog and prioritizing New Apps, Recommended and Business.  
{{<image src="/img/euc/hub-templates/AppCatalog_Sales.PNG" caption="AppCatalog Sales">}}
- and for Home tab, pointed the URL to Sales home page
{{<image src="/img/euc/hub-templates/home_Sales.PNG" caption="Home Tab Sales">}}

### Step 3: Create a Template
In this step, we will be creating templates for each personas with the feature set(s) that needs to override the Global setting. 

For the personas I've chosen, I will create two templates as follows:
- Knowledge Worker:  
{{<image src="/img/euc/hub-templates/template_KW.PNG" caption="Template Knowledge Worker">}}
- Sales:
{{<image src="/img/euc/hub-templates/template_Sales.PNG" caption="Template Sales">}}

### Step 4: Assign the Template 
On completion, you can assign the template to either smart groups or Access user group or both based on the experience you would like to provide
{{<image src="/img/euc/hub-templates/Assign.PNG" caption="Assignment">}}

{{< admonition type=note title="Note" open=true >}}
- iOS 
- Android 
{{< /admonition >}}

## Future: 
Experience Workflow
New Hire Onboarding 

