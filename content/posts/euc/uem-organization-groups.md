---
title: "What are Organization Groups in Workspace ONE?"
date: 2020-11-21T00:00:00-00:00
draft: false
categories: [UEM, EUC]
author: Partheeban Kandasamy
authorLink: "/about/"
tags:
- UEM
- EUC
---

What are Organization Groups? 

What are the best practices with how to set Organization Groups up for our use case? 

Can we insert an OG between OGs? Do we have the right setup? 

These are very common questions on Organization Groups which is a very foundational element within Workspace ONE UEM. Over the several years of working with Workspace ONE UEM (formerly AirWatch), no one has tried to provide a complete picture of what an Organization Group is. It is taken for granted and often misused! 

Previously, I've worked mostly with Enterprise Customers where UEM adoption is very mature, Organization Groups are pretty well laid out and is often already built out. Having recently transitioned to an EUC Architect, I now engage with a wide variety of Customers in different stages of Adoption and have got into many discussions and still get a lot of questions around the Organization Groups. Here is my attempt at bringing clarity on this topic.

## What are they?
An Organization Group is a way to organize information and form a structure for management within UEM. It can be considered similar to `folders within Operation Systems` or `Organization Units within Active Directory`. 

An Organization Group can have other Organization Groups as its child and Objects are anchored to an OG - Devices are enrolled at/to an OG. User accounts are created at an OG. Smart groups, Profiles, Apps are all created at an OG. I should note that in the past, new capabilities and features for reasons unclear were always made available in the "Customer type" OG but those are changing now. 

<image src="/img/uem/uem-organization-groups/og.png">

Here are the things that make up an Organization Group
- An Organization Group is defined by a 
  - **Name** - Arbitrary name of your choice
  - **GroupId (optional)** - GroupId has to be unique per environment and has to be present if a device is being enrolled in that Organization Group
  - **Type** - more on this [below](#types-of-organization-groups)
  - Country - If your Location/Organization Groups are based on Geos, then assign this to the right Country
  - **Locale** - Used to serve the right localized language within the UEM console
  - **Time Zone** - All the time shown within the UEM console are updated to reflect this timezone (or the timezone based on your user account)

From when the software was called AirWatch to now the Workspace ONE UEM, Organization Groups and Location Groups (behind the scenes) continue to remain one of those foundational design principle for a number of things..

### Object Anchor

- Users are created at an Organization Group and are allowed to enroll into an Organization Group. 
- Devices are enrolled in an Organization Group. 
- Originally used for assignments, the UEM has evolved to consolidating towards Smart Groups as the primary choice for assignments. The Smart Groups as wel
- Apps, Profiles, Products, Content are all anchored to the Organization Group 

### Multi-tenancy

It was designed as a solution to enable multi-tenancy within an AirWatch hosted environment. 

```
Global
├── Parent1
├── Parent2
├── Parent3
├── Parent4
├── Parent5
│   ├── Child1
│   └── Child2
│       ├── GrandChild1
│       ├── GrandChild2
│       ├── GrandChild3
│       ├── GrandChild4
```

- Permissions: An Admin at an OG is able to see all configurations and Objects(Devices, Profiles, Apps etc) at that OG and only those OGs that are its child
- Inheritance: A Device enrolled at a OG gets all the configuration that are assigned from its hierarchy above. 

### Configurations

Any configuration is created at an Organization Group and can be inherited or overridden at the OGs below. These configurations could span from 
- Settings such as Directory Services, Tunnel Configuration, ABM, APNS, Google Integration etc
- Objects such as Profiles, Apps, Products, Content

### Integrations

All integrations from On-Premise components such as UAG and ACC to integrations with other WS1 Components such as Access to Partner products such as [CheckPoint](https://www.checkpoint.com/downloads/products/sandblast-mobile-vmware-workspace-one-uem-solution-brief.pdf) or [Wandera](https://blogs.vmware.com/euc/2020/07/wandera-enriches-workspace-one-trust-network.html). 


## Types of Organization Groups

There are different types of Organization Groups, namely
- Global
- Customer
- Container
- Partner
- and others (even custom ones)

```
Global
├── Customer1
├── Customer2
├── Container1
│   ├── Container2
│   │   ├── Customer3
│   └── Customer4
│       ├── Container3
│       ├── Container4
```

They are generally setup like shown above and out of all the types available, the key ones to note are - 

### 1. Global
This is the default OG UEM ships with and can be considered the root Organization Group. 

> The general assumptions are that 
> - One does not do anything in this OG (outside of a few environment-wide settings - like site urls, CDN, file storage etc.)
> - You are always expected to always create a child OG to start configuration. 

### 2. Customer
This is the Organization Group type that is typically setup for a "Customer". 

> The general assumptions are that 
> - A Customer OG CANNOT have another Customer OG as its child
> - A Customer OG is usually where most integrations and configurations are setup
> - A Customer OG is where billing information and license consumptions are usually obtained

### 3. Container
This is, in my experience, usually every other OG type in UEM. 

> The general assumptions are that a Container OG can have a Customer OG as its child
> - Some Customers insert a Container b/w Global and Customer to give them some buffer for any changes
> - Go crazy with this and the other OG types

## Should I Create One?

A good rule of thumb is to be ruthlessly conservative and take a minimalist approach with Organization Groups. DO NOT create one unless they are absolutely needed to solve your use case. Here are some scenarios below for reference - 

### Definitely NOT
- **Assignments**: Use Smart Groups instead as they offer more flexibility

### Probably a Good Idea
- **Device Ownership**: You don't need to create OGs to separate devices by ownership types unless you have some Settings/Configurations (SDK policies for example) that are different for these device types
- **Device Type or Model**: You don't need to unless you have teams segregated by device types (iOS team vs Windows team)

### You should
- **GEOs or Locations**: Using OGs to mirror your Organization's geographical locations is a good idea as it also enables you to setup Administrators that can manage devices in that GEO
- **Agencies or Functions or Departments**: Using OGs to reflect how your Organization is setup by groups is a good idea as they let you setup different policies
- **Acquisitions**: Using OGs to migrate devices from M&A type of activities can give you the flexibility to setup policies differently and enabling a phased approach

## Take Away

**I could give you a few examples but they would be misleading!** As the software continues to evolve, what was once a great idea for setting up Organization Groups are probably not relevant anymore. They were setup based on the requirements, the design constraints and the capabilities available in the software at that point in time. Organization groups could be adapted a number of different ways and what worked for an Industry or a Customer might not work for others. 

Hope this gives you an understanding of what Organization Groups are, why they exists and how to use them. The bottom line is to not get too constrained by Organization Groups and conversely don't get carried away with Organization Groups. 

Please provide any feedback that you have by leaving a comment and share this post using the social media icons shown below. That's all folks! 