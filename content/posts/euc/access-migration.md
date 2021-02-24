---
title: "Fling: Migration tool for WS1 Access"
date: 2021-02-23T23:58:00-00:00
draft: false
categories: [ACCESS, EUC]
tags:
- Access
- APIs
- Migration
- Fling
lightgallery: true
featured: true
---

## Migration?...Migraine?...

When it comes to migration scenarios, the most involved and difficult migration within the EUC lineup (that I can think of) is the WS1 Access tenant migration. It is equivalent to adopting a new IDP solution as you'll have to re-federate all the apps in order to successfully complete a migration.  

And this is case regardless of where you are trying to move from - 

### **On-Premise to SaaS**
 - As Cloud is picking up steam and as features are developed with a cloud-first mentality, tt makes sense to offload the management of the Access Service to VMware
 - This scenario is also getting more common as Access and Hub Services are FedRAMP certified as of CY Q4 2020

### **SaaS to SaaS**
 - This is a not very common use case but few situations demand them nonetheless predominantly due to issues arising from high customization entertained early in the product adoption lifecycle
 - Another use case for is populating a UAT or a DEV tenant with Apps from another tenant

------------

## Path to Migration

First and foremost, a migration is possible as documented in [@williamsmt blog](https://blog.virtualprivateer.com/workspace-one-cloud-migration/) which provides a detailed walk through of the entire migration approach. Where this process becomes painstakingly long is also highlighted here

{{< admonition type=info title="Excerpt from VirtualPrivateer" open=true >}}
Perform steps 5-10 for each web application that should be created to mirror app availability in the existing WS1 Access on-premises environment. For those who are migrating in one phase, you can use my colleague Chris Halstead’s Fling for automating the majority of this process.
{{< /admonition >}}

{{<image src="/img/euc/access-migration/fling-chris.png" caption="Access Migration Tool - Logo">}}

As I looked deeper into Chris Halstead's Migration/Backup Fling {{% twitter profile="chrisdhalstead" %}}, I noticed it could be used for two main use cases
- it is an excellent resource for backing up and restoring applications into a single Access tenant
- for a pure migration scenarios, it could also be leveraged to achieve the migration of WebLinks.

However, when it comes to Federated Apps, the Fling falls short as it automated exporting and importing of the apps. To automate and ease the migration of Federated Apps, the Fling needed to acquire the launch url of the Federated App from the source tenant along with other meta-data (image for example) to create a web link in the destination tenant.

------------

## Migrating Federated Apps

So, is there a better way to migrate Apps that are directly federated with Access? 

Yes, here are some options
- If you don't have a lot of Apps federated with Access, the "get it done" approach is to take some downtime and re-federate all the apps to the new tenant. 
- If you have a lot of apps however, it is going to be painstaking to setup some time with your App owners to re-federate all the apps at the same time...and this is where the fling comes in handy

------------

## WS1 Access Migration Fling

I wrote a C# Windows Application (released as a Fling) which is inspired from and provides the missing feature on Chris Halstead's Fling for the federated apps. 

Fling URL - https://flings.vmware.com/workspace-one-access-migration-tool 

{{<image src="/img/euc/access-migration/app-logo-color.png" caption="Access Migration Tool - Logo">}}

------------

## How can you use it? 

First, download the App from flings.vmware.com. The fling is a C# Windows Forms app app which you can extract into a folder to then run the application. 

As you launch, you'll see two sections - to the left is the section for source tenant and to the right will be the section for destination tenant. The goal is to move or copy objects from the source to destination. 

{{<image src="/img/euc/access-migration/app-launch.png" caption="Access Migration Tool - First Launch">}}

> Remember this is a fling and I've prioritized utility over experience. 

### Login 
- Provide the https url for your tenant - for example - `https://access.lab.com`
- Provide the credentials for your tenant - this should be the local system domain account with administration privileges 
- Clicking on Login with the proper credentials should log you in 

### Get Categories and Apps
- Upon logging in, the `Get Apps` and `Get Categories` features are "unlocked"
- Clicking on `Get Categories` for the tenant that you are logged in, will fetch all the Categories on that tenant. The categories will be listed in the dropdown right next to the `Get Categories` button
- Choosing the type of App (SAML, Weblink etc) and Clicking on `Get Apps` for the tenant that you are logged in, will fetch all the corresponding type of App on that tenant. The Apps will be listed in the table at the bottom of the App. 

{{<image src="/img/euc/access-migration/app-view.png" caption="Access Migration Tool - Viewing Categories and Apps">}}

### Migrate Categories
To facilitate a smooth migration for apps and to retain all the category to app mapping, you'll want to copy the Categories from the source tenant to the destination tenant. 
- To Migrate Categories, please login to the destination tenant as well
- Once you've logged in to the destination tenant, the `Copy Categories` button will be activated
- Click the `Copy Categories` to migrate all the categories from the source to the destination tenant

> Categories are unique by name and in case of conflicts only the unique ones are created in the new tenant (*not sure about how upper vs lower case is handled*)

### Export Apps
- Once you are able to view the Apps from the source tenant, the option to `Export Apps` are also enabled. This action will prompt you to choose a folder to export the App metadata. 
- Upon choosing the Folder location, the apps that were selected will be exported as zip files (one zip per app)
- The Apps are exported into appropriate folders that represent their categories (Web App, SAML2.0, WSFED, SAML1.0)
- Exporting the Apps will further enable the `Copy Web Apps`, `Link SAML2.0 Apps`, `Link WSFed Apps`, `Link SAML1.0 Apps` and `Copy App to Category Assignment` actions that can be performed on the destination tenant

{{<image src="/img/euc/access-migration/app-export.png" caption="Access Migration Tool - Exporting Apps">}}

### Import Apps
This is the crux of this whole Fling which enables you to migrate Apps to the destination Access tenant

#### Migrate Web Links
- Clicking on `Copy Web Apps` will prompt for the folder path that you can choose to import the apps from. 
- The Fling will try to find a web apps folder and import the apps into the destination tenant

#### Migrate Federated Apps (SAML2.0, WSFED, SAML1.0)
- Clicking on `Link SAML2.0 Apps`, `Link WSFed Apps`, `Link SAML1.0 Apps` will prompt for the folder path that you can choose to import the apps from. 
- The Fling will try to find apps from the appropriate folder and import them into the destination tenant

#### Copy the App-Category mapping 

> PRE-REQs - All the categories must be present in the destination directory

- Clicking on `Copy App to Category Assignment` will attempt to match the app categorization in the source tenant to the destination tenant

### Video Walkthrough

{{% youtube QdIEDM93c-M %}}

I've built a lot of tools but this is a monumental occasion as this is my first Fling (that was born out of a Customer's need to migrate Access tenant) that I've authored and released to the public. Please go try it out and let me know how you like it! Shout out to everyone that supported by testing, providing test tenants and by sharing their feedback. Hoping you'll take advantage of the WS1 Access Migration Tool for any of the use cases that I've highlighted above! 

Happy Migrating! 