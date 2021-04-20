---
title: "Update 1: Migration tool for WS1 Access"
date: 2021-04-14T00:00:00-00:00
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

This is a quick post covering an exciting update to the Workspace ONE Access Migration Tool. If you are interested in learning more about the tool, please check out these resources - 
- [Why was this Migration tool built?](/post/access-migration/)
- [How can you use it?](/post/access-migration/#how-can-you-use-it)
- [Download the Migration tool](https://flings.vmware.com/workspace-one-access-migration-tool)

{{<image src="/img/euc/access-migration/app-logo-color.png" caption="Access Migration Tool - Logo">}}

As a quick recap, the fling allows easy migration of Apps (excluding virtual apps) from one WS1 Access tenant to another (both on-prem to SaaS as well as SaaS to SaaS) by preserving the user experience as much as possible while reducing the migration effort by automating these major tasks - 

- Migrate the complete list of Categories from one tenant to another
- Export Apps from one tenant and Import them into another (preserving the icons)
- Migrate the App-Category associations

## Copy the App-Group mapping 

One of the missing features that this update aims to solve is the ability to migrate the App Entitlements and more specifically looks to migrate the Group Entitlements for the Apps. 

{{<image src="/img/euc/access-migration/app-group-migrate.png" caption="Access Migration Tool - Entitlement Migration">}}

{{<image src="/img/euc/access-migration/app-group-migrate-done.png" caption="Access Migration Tool - Entitlement Migration">}}

### Why only Groups?

The first question that you're going to have is - why is there a restriction to migrate just the Group based assignment? The answer lies in the inherent interest to reduce complexity.

1. Based on experience, Apps within Workspace ONE Access that are assigned to users directly are a significantly low population. Moreover assigning apps to Users is not a very scalable process for IT organizations as well. 
2. Group and User lookups are very costly operations (more below) - the group/user will have to exist in both tenants and the APIs (as far as I can tell) do not provide a way to lookup these entities by their ObjectGUID. 


### How does it work?

To incorporate this feature, there were some fundamental assumptions that I had to start with 

#### Assumptions
1. First assumption is that the App exists in both tenant - App is matched by their name and before attempting to Migrate assignments, you should always hit "Get Apps" to ensure that the App is showing up in both the lists (on the source and destination)
2. Second assumption is that the group exists in both tenants - make sure you have configured the new tenant with the right Connectors and have the groups synced. 

#### Implementation Details
For those looking to understand more details about how it works, here is the logic used

- Build a master list of groups to match the Groups across two tenants
  - Generate a master list of groups from source tenant (entitlements for groups use the tenant's UUID of the group, i need a common identifier that this group shares in the other tenant such as ObjectGUID/externalID)
  - Generate a master list of groups from destination tenant (entitlements for groups use the tenant's UUID of the group, i need to reverse lookup the group using the objectGUID/externalID to find the UUID)
- For every selected app
  - Get the group entitlements (if no entitlements exists, we skip this app)
  - For every entitled group, lookup the group in source tenant to find the external id
  - Check to see if that external id exists in the destination tenant. If yes, get the UUID for the group and add to entitlements for that app. If no, we skip this group entitlement for the app

- Upon completion, a Message Box shows the status of the migration
  - how many succeeded, 
  - how many failed (for unknown reasons), 
  - how many apps were skipped because they were not found in the destination tenant and 
  - how many apps assignments were skipped because the groups didn't exist in the destination tenant

I'm hoping you'll take advantage of the WS1 Access Migration Tool to save yourself some time. If you found this useful, please share your success stories and feedback! Shout out to everyone that supported by testing, providing test tenants and by sharing their feedback. 

Happy Migrating! 