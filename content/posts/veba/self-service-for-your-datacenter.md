---
title: "Enabling Self Service for your DataCenter — Part I"
date: 2020-10-07T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
---

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/self-service-for-your-datacenter-part-i](https://medium.com/enabling-self-service-for-your-datacenter-part-i-c550176b6c1c)
{{< /admonition >}}

It doesn’t have to be **Business vs IT** anymore! Either sides’ needs while well-intentioned may seem to be at conflict with each other..

**Business** aims to innovate and promote new capabilities for their consumers with the goal of improving services delivered or user experience to retain existing users or acquire new users.

**IT** as true partners to the Business want to provide the right platform and infrastructure but need to ensure that they guaranteeing the right security guidance, conformance to standards and best practices.

A key area that often exacerbates this divide is how interaction between Business and IT teams are facilitated. Traditionally and in a majority of existing Enterprises, Ticketing Systems are often the preferred way for Business to engage IT. Once a request is placed, it is then subjected to an elaborate Change Management Approval process before initiating a project requested by Business. Such processes ensure thoroughness but stunts innovation agility.

Modern Cloud Platforms have challenged that approach by enabling anyone with just a credit card to spin up Servers, Services or Desktops anytime — leading to the rise of Shadow IT. What if we bring these cloud-like capabilities to the SDDC? [F. Gold](https://medium.com/u/d568d91705bd?source=post_page-----c550176b6c1c--------------------------------) and I recently delivered a VMworld Session where we built a demo of doing just that!

> Arm Yourself with Event-Driven Functions and Reimagine SDDC Capabilities [HCP1404] — [http://bit.ly/pksession](http://bit.ly/pksession)

The idea stemmed from my work with the [VMware Event Broker Appliance](https://vmweventbroker.io/) (VEBA). VMware Event Broker Appliance deployed with [OpenFaaS](https://www.openfaas.com/) provides the ability to deploy functions which can be triggered in two different ways —

*   **Event Driven** — example function that get triggered on an event to make a POST request to an HTTP endpoint is [available here](https://medium.com/@pkblah/a-function-for-all-rest-apis-403b4fdf15e8?source=friends_link&sk=fc845deda60e4d11c022374c4b21ee0a) (works with Pagerduty, Slack, ServiceNow, ServiceDesk, Jira etc)
*   **Command/User Driven** — enabled through the function’s HTTP endpoint as explained in OpenFaaS’s blog [here](https://www.openfaas.com/blog/stateless-microservices/)

As [Team #VEBA](https://vmweventbroker.io/#team-veba) continues to look for ways to stretch the boundaries around capabilities unlocked through VEBA, I wanted to partner up with [F. Gold](https://medium.com/u/d568d91705bd?source=post_page-----c550176b6c1c--------------------------------) to leverage the event-driven capabilities as well as incorporate the command-driven aspect of a deployed function to deliver a true self-service experience for Business Units.

## So, what does the Self-Service app do?

As we started putting our heads together and thinking of a good usecase to explore both the event-driven and command/user driven capabilities, the idea of a Slack bot came to mind along with some auto-remediation of issues.

Here are the questions that we sought out to answer

For the **Command/User driven** use case, What if IT enables Business Stakeholders to interact with VMware SDDC through a Slack command to —

*   **Create a VM** — This can be useful in the early stages of enabling self-service. IT can control the size of the VM spec by possibly starting with their standard VM Spec or maybe even a lower Spec
*   **Clone a VM** — This can enable horizontal scaling of their Application or troubleshoot configuration issues by cloning their existing VMs
*   **Clone from a VM Template** — When Business teams standardize on a VM Spec and Template, IT could create a VM Template as specified by Business and validated by security. This template can be cloned by the user for their Application deployment
*   **Power cycle VMs** — At any given time, there may be a need to power cycle VMs (outside of the OS controls available). These controls can be provided to the user and also giving them the power to utilize resources efficiently
*   **Delete VM** —IT can offload resposible clean up of SDDC resources by enabling the BU to delete VMs on demand

For the **event-driven** use case, what if certain crises are handled automatically thus increasing operational efficiency —

*   **Vertical Scaling a VM** — Not everyone has a clear understanding of what their VM’s spec is going to be. More often than not, to play safe, the request for VM Spec has added buffer and is unreasonably sized. What if when a VM is running low on CPU or Memory resources, a function is triggered to auto increase the VM Spec (by 1 CPU upto a predetermined max value and by 2Gb upto a predetermined max value). This will enable Business teams to start with a low or appropriately sized VM and give the assurance that VMs can scale up to handle any future loads
*   **Auto Storage DRS** — When a host datastore is running out of space, a function can be triggered to move the VM to a different datastore.

{{}<youtube iJ39aMVvMR8>}

## Architecture

To make this App a reality, we are going to build three main components

First, the **Gateway function** called `**iaasgw**` written in `**python**`**—** This function is going to recieve the Slack bot command and based on the command, triage to a set of VM Lifecycle functions.

> This function was written in Python because of my familiarity with Python.

Second, the **VM Lifecycle functions** written in `**PowerCLI**`— These functions only respond to the Gateway function and will help perform a specific action on a VM within vCenter. There are also a couple of Utility functions that i’ve used for debugging and to query vCenter.

> These functions were first attempted to be implemented in Python but I found the PowerCLI an easy and efficient way to interact with vSphere.

Lastly, the **Event-driven crisis remediation functions** written in `Go`— These functions will be triggered when an Alarm status changes to`red`.

> These functions were written in Go purely because Frankie,our Go expert, was exploring and implementing these usecases.

{{<image src="https://miro.medium.com/max/3012/1*l2CTz_sUAjRmWugTV_952A.png" caption="Self-Service App Architecture with VEBA (depoyed with OpenFaaS)" >}}

Now, let’s look at these components in detail.

## The Gateway Function (written in python)

Let’s start by creating the Gateway function which is the entry point for Slack Bot.

You can find the source code for this function on the [pksrc/vebafn](https://github.com/pksrc/vebafn/tree/master/vm-self-service-app) repository here — [https://github.com/pksrc/vebafn/tree/master/vm-self-service-app/python/iaasgw](https://github.com/pksrc/vebafn/tree/master/vm-self-service-app/python/iaasgw)

### Slack App Setup

Slack bot command setup require a publicly available HTTP endpoint. For security reasons and for most third party integrations, there is most likely a requirement to have a secure endpoint also.

{{<image src="https://miro.medium.com/max/1100/1*AnUCLWLuimMZdP0PQfQAlA.png" >}}

For my demo and as previously documented [here](https://medium.com/@pkblah/publicly-trusted-tls-for-vmware-eventing-platform-6c6f5d0a14fb?source=friends_link&sk=2801b0b75bdc1bddfd1fcc7fa4d28d67), I used Let’s Encrypt to get a publicly trusted TLS certificate. I also needed a public IP for which i used NoIP to map the Dynamic IP from my ISP to a DNS.

> Publicly trusted TLS for VMware Event Broker — [here](https://medium.com/@pkblah/publicly-trusted-tls-for-vmware-eventing-platform-6c6f5d0a14fb?source=friends_link&sk=2801b0b75bdc1bddfd1fcc7fa4d28d67)
> 
> Dynamic DNS solution with NoIP — [here](https://www.noip.com/)

> _Pro Tip:_ If you are setting up a Slackbot with OpenFaaS functions, make sure to use the async-function path for the deployed function. For example if your deployed function URL is `[https://pdotk.lab.net/function/veba-echo](https://pdotk.ddns.net/function/veba-echo)` use `[https://pdotk.lab.net/async-function/veba-echo](https://pdotk.ddns.net/function/veba-echo)` as this ensures that an acknowledgement is immediately sent back to Slack and thus giving you the ability to do any processing asynchronously.

### Receive payload from Slack

Once the Bot is setup and deployed to a channel, when a user invokes the Slack slash command `/iaas`, Slack makes a HTTP request to the `Request URL` specified during the App setup. The function would receive the following request parameters as explained [here](https://api.slack.com/interactivity/slash-commands) and as shown below

```
###  
# token = 'pXxmdXfxxxxxxxxx4mslg'  
# team_id = 'T9DXXXD7Z'  
# team_domain = 'pkslack'  
# channel_id = 'C017763BZGX'  
# channel_name = 'vmworld2020'  
# user_id = 'U9BXXXEQ1'  
# user_name = 'partheeban.kandasamy'  
# command = 'echo'  
# response_url = 'https://hooks.slack.com/commands/T9DiY7'  
# trigger_id = '1258627504981.319069523.979f734485f07cc27'  
# text = "createvm"  
##
```

> _Pro Tip:_ If you are working on a new integration or even when developing a new VEBA event-driven function and need to check the event payload, deploy any of the echo functions (here is the python [version](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/master/examples/python/echo)) to get a sense for the payload that the function is going to receive.

### Verification and Acknowledgement

The first couse of action, immediately after recieving the payload from Slack, is to verify the authenticity by following the set of steps descibed in Slack’s documentation [here](https://api.slack.com/authentication/verifying-requests-from-slack). This ensures messages are not tampered during transit or protects against replay attacks.

> _Pro Tip:_ With OpenFaaS functions, all the HTTP headers are made available within the function (container) as an environment variable prepended with `Http_` and all the hyphens replaced with underscores. For example `X-Slack-Signature` in the HTTP request header would be available as an environment variable `Http_X_Slack_Signature`.

Secondly, to ensure good user experience, we are going to respond back to the user with an acknowledgement. This acknowledgement will be sent to let the user know what command was invoked by making a POST request to the `response_url` that we got from Slack.

### Process Commands

Now, all that is left to do is to process the commands. For this, we’ll be triaging by making a HTTP POST request to a set of VM Lifecycle functions that will be deployed in the same OpenFaaS cluster.

> _Pro Tip:_ These functions will be available through the endpoint `http://gateway.openfaas:8080` within the kubernetes cluster. For example `_https://pdotk.lab.net/function/veba-echo_` _would be available at_ `_http://gateway.openfaas:8080/function/veba-echo_`

To ensure security, we’ll be adding a secret key to the Slack payload which will be sent with the POST request to the VM Lifecycle function. This secret will be shared with the VM Lifecycle functions to ensure the request is indeed coming from the Gateway function.

Based on the command recieved from the user, the right VM function will be invoked as shown below…

```
if('**echo**'):  
call [http://gateway.openfaas:8080/async-function/**powercli-echo**](http://gateway.openfaas:8080/async-function/powercli-echo')  
with json=slack_payload + shared_keyelif('**spawn**'):  
call http://gateway.openfaas:8080/async-function/**powercli-createvm**'  
with json=slack_payload + shared_keyelif('**clonetemplate**'):  
call http://gateway.openfaas:8080/function/**powercli-vmclonetemplate**  
with json=slack_payload + shared_keyelif('**clone**'):  
call http://gateway.openfaas:8080/async-function/**powercli-clonevm**  
with json=slack_payload + shared_keyelif('**poweron**'):  
call [http://gateway.openfaas:8080/async-function/**powercli-poweronvm**](http://gateway.openfaas:8080/async-function/powercli-poweronvm)with json=slack_payload + shared_keyelif('**poweroff**'):  
call [http://gateway.openfaas:8080/async-function/**powercli-poweroffvm**](http://gateway.openfaas:8080/async-function/powercli-poweroffvm)  with json=slack_payload + shared_keyelif('**reboot**'):  
call http://gateway.openfaas:8080/async-function/**powercli-rebootvm**  
with json=slack_payload + shared_keyelif('**nuke**'):  
call http://gateway.openfaas:8080/async-function/**powercli-deletevm**  
with json=slack_payload + shared_keyelif('**transform**'):  
call http://gateway.openfaas:8080/async-function/**powercli-setvm**  
with json=slack_payload + shared_keyelif('**invoke**'):  
call http://gateway.openfaas:8080/async-function/**powercli-danger**  
with json=slack_payload + shared_keyelse:  
#Make a POST request to Slack response URL with an  
**ERROR** that the command was not found
```

Now, let’s take a look at the VM Lifecycle functions in the next part!

## The VM Lifecycle Functions (written in PowerCLI)

These are the easiest functions of the lot to implement. We have set of functions implemented in PowerCLI that take a specific action within vCenter.

You can find the source code for this function on the [pksrc/vebafn](https://github.com/pksrc/vebafn/tree/master/vm-self-service-app) repository here — [https://github.com/pksrc/vebafn/tree/master/vm-self-service-app/powercli](https://github.com/pksrc/vebafn/tree/master/vm-self-service-app/powercli/vmlifecycle)

These are architected to facilitate business logic addition or modification without impacting other VM functions or commands. For example, you could restrict the power cycle actions to be performed on VMs within a certain Host only, or the Create VM commands could be configured to add VMs to a certain host only. The high-level implementation logic for these functions is provided below

```
**Psuedo Code**  
1. Process function secrets or configs - this function needs the vCenter Credentials2. Process the payload received from the Gateway3. Validate that the request is indeed from the Gateway function before allowing any critical functionality - verify the presence of the shared secret4. Connect to vCenter Server and Perform the intended change such as poweron, poweroff etc.. 
```

Here is the breakdown of all the functions, their functionality and how you can invoke them once they have been deployed.

### Utility — Echo the payload from the Gateway

The echo function helps with troubleshooting by printing to System Out the payload received by the function.

```
**command: /iaas echo <type something here>  
image**: pkbu/powercli-echo
```

### Create a VM:

Creates a VM of a certain specification which is configured and hardcoded to be 1 CPU, 128MB Memory and 128MB HDD. This is obviously not a realistic value for most scenarios and will have to be adjusted accordingly.

```
**command: /iaas spawn <vmname>  
image**: pkbu/powercli-createvm
```

### Clone a VM (not covered in demo)

Creates a clone of a specific VM — I have a “TestVM” in my LAB which will be cloned. In a real world scenario, this could be a clone of a PROD VM to replicate issues or scale out services installed on a VM.

```
**command: /iaas clone <vmname>  
image**: pkbu/powercli-clonevm
```

### Clone a VM from Template

Creates a clone from a VM Template. Once you have an idea of the VM Spec and the Operating System that is needed, it can be made available as a VM Template which can be cloned at will by the Slack user.

```
**command: /iaas clonetemplate <vmname> <templatename>  
image**: powercli-vmclonetemplate
```

### Power Cycle functions

A set of useful powercycle functions for a VM. In a real world scenario, it helps users be responsible about managing available resources as well as giving them the ability to powercycle VMs as a troubleshooting step at will.

```
**command: /iaas poweron <vmname>  
image**: [pkbu/powercli-poweronvm](http://gateway.openfaas:8080/async-function/powercli-poweronvm)**command: /iaas poweroff <vmname>  
image**: pkbu[/powercli-poweroffvm](http://gateway.openfaas:8080/async-function/powercli-poweroffvm)**command: /iaas reset <vmname>  
image**: pkbu[/powercli-rebootvm](http://gateway.openfaas:8080/async-function/powercli-rebootvm)
```

### Remove a VM

Helps delete the VM from vCenter. Another helpful function that promotes responsible utilization of available resources. In real world, you must use caution and put in gaurd rails to prevent deleting VMs that must not be deleted.

```
**command: /iaas nuke <vmname>  
**- **image**: pkbu[/powercli-deletevm](http://gateway.openfaas:8080/async-function/powercli-deletevm)
```

### Transform a VM (not covered in demo)

This function helps change the VM hardware spec at will by the Slack user. This wasn’t covered in the VMworld demo as this was handled through an event-driven scenario that will be covered in the Go function writeup.

```
**command: /iaas transform <vmname> <templatename>  
image**: pkbu/[powercli-setvm](http://gateway.openfaas:8080/async-function/powercli-setvm)
```

### Utility — Run a PowerCLI command on vCenter _(use caution)_

This is a helpful **_yet dangerous_** function as it executes a PowerCLI command as-is against a established vCenter connection and the response (if any) is sent back to Slack. I’ve used this to query the status of a VM at any given point in time as shown below.

```
**command: /iaas invoke get-vm -Name veba-test-vm| fl Id, Name, PowerState, NumCpu, CoresPerSocket, MemoryMB, VMHost | Out-string  
image**: pkbu[/powercli-danger](http://gateway.openfaas:8080/async-function/powercli-danger)
```

## Function Deployment

We’ve provided detailed steps on how to deploy these functions [here](https://github.com/pksrc/vebafn/tree/master/vm-self-service-app) in the Readme. You should be able to tweak, deploy these functions and enable VM Self Service in your organization right away!

We’ve covered a lot in this article and in Part II of this topic [F. Gold](https://medium.com/u/d568d91705bd?source=post_page-----c550176b6c1c--------------------------------) will cover the the Event-driven Remediation functions.

> [Enabling Self Service for your DataCenter — Part II](https://medium.com/@codeaurix/enabling-self-service-for-your-datacenter-part-ii-943b670d0614)

This required a lot of effort to become a reality! Please let us know (tweet @[pkblah](https://twitter.com/pkblah)) how you’ve used this or liked what we’ve made available.

Happy Eventing!
