---
title: "Integrating vCenter with PagerDuty"
date: 2020-04-30T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
featuredimage: https://miro.medium.com/max/3040/1*I37x-vfp3ry3Y4O3f_Q-Kg.png
featuredImagePreview: https://miro.medium.com/max/552/1*I37x-vfp3ry3Y4O3f_Q-Kg.png
---

<!--
<img alt="Image for post" class="fg ev er gk w" src="https://miro.medium.com/max/3040/1*I37x-vfp3ry3Y4O3f_Q-Kg.png" width="1520" height="654" srcSet="https://miro.medium.com/max/552/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 276w, https://miro.medium.com/max/1104/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 552w, https://miro.medium.com/max/1280/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 640w, https://miro.medium.com/max/1456/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 728w, https://miro.medium.com/max/1632/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 816w, https://miro.medium.com/max/1808/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 904w, https://miro.medium.com/max/1984/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 992w, https://miro.medium.com/max/2000/1*I37x-vfp3ry3Y4O3f_Q-Kg.png 1000w" sizes="1000px"/>
-->

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/integrating-vcenter-with-pagerduty](https://medium.com/integrating-vcenter-with-pagerduty-241d3813871c)
{{< /admonition >}}

Uptime and Reliability is more important now than ever during these times when Technology and Infrastructure is enabling us fight a global pandemic with work from home policies. It is no wonder that eyes lit up when you say PagerDuty and vCenter integration!

I’m going to explore how you enable this integration to automatically trigger a PagerDuty incident the minute vCenter detects something bad happens to your infrastructure!

## Deploy VEBA

Let’s start with deploying [VMware Event Broker Appliance (VEBA)](https://flings.vmware.com/vcenter-event-broker-appliance) which provides a event-driven Function as a Service running on a single node Kubernetes cluster delivered to you with the convenience of an appliance. Deploying the appliance takes less than 30 minutes provided you have your environment information readily available (Networking, Proxy and vCenter service credentials).

## PagerDuty Setup

{{< admonition type=warning title="Disclaimer" open=true >}}
Skip over this if you are familiar with the first time setup process and also I’m no PagerDuty expert - There may be more than one way to do this or many considerations in setting this up for your Organization
{{< /admonition >}}

{{<youtube 1iA4P-rLzg8 >}}

We are going to be making an API call to PagerDuty Events API to `trigger` an incident for a service. While most of you might have your Pagerduty configured, i’m going to provide the steps that i used to guide those that are setting it up for the first time as a POC or for testing.

*   First I signed up for a developer PagerDuty account

> Sign up for your PagerDuty Developer account [here](https://developer.pagerduty.com/sign-up/)

*   Once you have your account activated or in your existing setup, you first need a team or user created, this can be done by going to Configuration →Team or Configuration →Users to create one or both

> Instructions to create users are found [here](https://support.pagerduty.com/docs/users) and to create teams are found [here](https://support.pagerduty.com/docs/teams)

*   Now that you have a team/user created, proceed to create an Escalation policy by going to Configuration → Escalations to define what happens when an incident is triggered

> Instructions to create an escalation policy can be found [here](https://support.pagerduty.com/docs/escalation-policies)

*   Once you have your escalations created, you are now ready to create your Service which represent an application, component, or team you wish to open incidents against.
*   For our Service, we are going to setup the Integration to use PagerDuty Events API v2. This will give us the Integration key for us to use later within our function that we are going to deploy.

> Instructions to create a service can be found [here](https://support.pagerduty.com/docs/services-and-integrations)

*   To finish up and optionally, I also enabled the Slack v2 extension for my service which allows Pagerduty to send a Slack Notification when an incident is triggered. This integration also allows you to resolve an incident directly from Slack as well! Pretty neat!

> Instructions to set up the Slack and PagerDuty integration can be found [here](https://support.pagerduty.com/docs/slack-integration-guide)

**Alternatively, if you are looking to directly integrate your vCenter with Slack and send notifications, you can do that directly using our pre-built function that’s available** [here](https://github.com/pksrc/vcenter-event-broker-appliance/tree/restapi-fn/examples/python/invoke-rest-api)

## **Integrating with vCenter**

Now that you have VEBA deployed as well as have PagerDuty setup for our integration, let’s get going with deploying our function which is readily available [here](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/python/trigger-pagerduty-incident). Find the summary of deployment steps below.

### 1.  **Clone VEBA repository**

```
git clone https://github.com/vmware-samples/vcenter-event-broker-appliance  
cd vcenter-event-broker-appliance/examples/python/trigger-pagerduty-incident
```

### 2. **Edit the configuration file**

Change the configuration file `pdconfig.json` to add your integration key

```
{  
    "routing_key": "<replace with your routing key>",  
    "event_action": "trigger"   
}
```

### 3. **Create the secret**

```
export OPENFAAS_URL=https://VEBA_FQDN_OR_IP  
faas-cli login -p VEBA_OPENFAAS_PASSWORD --tls-no-verify   
  
# now create the secret  
faas-cli secret create pdconfig --from-file=pdconfig.json --tls-no-verify
```

### 4. **Deploy the function**

```
faas-cli deploy -f stack.yml --tls-no-verify
```

That’s it! The function by default is triggered by the VM Power Off/On Event and will immediately trigger a PagerDuty incident for our Service.

If you are looking to customize the function, the detailed deployment steps provided in the `README.md` from the repository linked below should have all the information you are looking for.

{{< admonition type=notes title="Repository Information" open=true >}}
[vmware-samples/vcenter-event-broker-appliance](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/python/trigger-pagerduty-incident)
{{< /admonition >}}

Hopefully, this easy integration, enabled by VEBA, helps with your Organization’s SDDC Incident Management with PagerDuty.

Let us know what other integrations you’d like to see and Happy Eventing!
