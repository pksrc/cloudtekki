---
title: "Writing your first Serverless Function"
date: 2020-04-23T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
---

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/writing-your-first-serverless-function](https://pkblah.medium.com/writing-your-first-serverless-function-23508cb4ea11)
{{< /admonition >}}

A function is a unit of execution in the Serverless world that does one thing and one thing really well. With the current product [VMware Event Broker Appliance](https://github.com/vmware-samples/vcenter-event-broker-appliance) (VEBA) that i’m managing, we aim to provide a simple solution that provides a way to execute your functions driven by vCenter events. This is a significant capability that exposes a plethora of integrations and allows seamless automation opportunities for a VMware SDDC customer.

As the Product Manager, I wanted to take the opportunity to write a function not only to contribute and help the team but also as an effective way to understand one of our persona that we are targetting — developers and especially the ones that are new to programming. This exercise of building a function gave me several insights as well as enabled me to learn more about our product and the underlying tech capabilities and constraints. Here is my account of my experience and learnings!

{{<image src="https://miro.medium.com/max/1920/0*mUvkWiBfhKiV55L9" width="25%" class="imageright" >}}

## Development Setup

**OS**: macOS Catalina

**IDE**: VisualStudio Code

**Version Control:**Git

**Language** **of choice**: Python (no prior professional experience but have dabbled with it a few times)

**FaaS Platform:** VEBA deployed with [OpenFaaS](https://www.openfaas.com/). Alternatively, you could have OpenFaaS running on Kubernetes w/ VMware Event router container deployed to get the same functionality

**vCenter:** Required since our functions will be driven by events generated within the SDDC. I have a homelab that i can work with for testing purposes.

## Function Execution Flow

VMware Event Broker Appliance deployed with [OpenFaaS](https://www.openfaas.com/) makes it easy to write functions and before we start writing the function, it’s good to get a visualization of what the function execution flow look like.

The VMware Event Router streams the events from vCenter, packages those events into CloudEvent spec and invokes the appropriate function when it sees the right event.

The Event is passed to OpenFaaS as a http request which then invokes the function passing the event down as a std input argument which you can read and start building your business logic.

{{<image src="https://miro.medium.com/max/1920/1*KW6s2ZISEwHyymX9CQkZ7g.gif" caption="Function execution flow with vCenter Event Broker Appliance">}}

OpenFaaS provides readily available templates that you can use to get started and also has [workshops](https://github.com/openfaas/workshop) that you can also use as a resource.

## Writing the Function

For my first function, i’m going to trigger a Pagerduty incident upon an event in vCenter. After some quick research, i was able to setup Pagerduty with a developer account, nailed down the API that i want to call and also decided on using the VM Power On/Off event to trigger an incident.

### Hello Serverless

Now before jumping into the code itself, i spent some time putting together a minimal structure to get the function working and doing something simple— such as printing the event passed down to my Function. This is what the file structure looks like..

``` {.jr .js .jt .ju .jv .ml .mm .mn}
├── README.md      > A descriptive documentation for your function
├── handler        > Folder for your scripts
│   └── handler.py > Python file for your core function logic
└── stack.yml      > Descriptor file for OpenFaaS deployment
```

A version of this is available within the examples folder of the VMware Event Broker Appliance github repository [here](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/python/echo).

{{< admonition type=note title="Note" open=true >}}
I have also made a templatized version of this available [here](https://github.com/pksrc/vebafn/tree/master/python/sm-python-fn) for you to get started quickly and build on top of this.
{{< /admonition >}}

```bash {.nc .nd .ne .nf .ng .ml .mm .mn}
git clone git@github.com:pksrc/vebafn.git vebafncd vebafn/python/sm-python-fn
```

Now that you have this folder cloned, let’s get started..

{{<image src="https://miro.medium.com/max/1350/1*t1VlZ58bC8uUHIFNX16wvg.png" caption="stack.yml — primary descriptor for our function" class="imageright">}}

{{< admonition type=note title="Note" open=true >}}
I’ve provided screenshots below due to formatting reasons, all the code is available in the git repository [here](https://github.com/pksrc/vebafn/tree/master/python/sm-python-fn)
{{< /admonition >}}

**stack.yml** — this is the primary descriptor file which we are going to use to deploy the function. I made modifications as my environment required. A sample shown below..

{{<image src="https://miro.medium.com/max/1252/1*1d5sJTmK0RxM8J8w_Gd4Yw.png" caption="handler.py — parses the event and pretty prints as json to stdout" class="imageright">}}

**handler.py**— this is the core python file that should have the business logic for my function. This is what it looks like to start with..

**README.md**— this file contains the documentation describing the function purpose as well as steps to deploy the function

With this minimal code, I proceeded to deploy this code to OpenFaaS on VEBA. You can do this by using the deployment steps provided in the README.md file [here](https://github.com/pksrc/vebafn/tree/master/python/sm-python-fn)

### Here we go..

Once I got comfortable with the deploying this simple function and seeing it execute, i proceeded with making the next iteration of changes and make our API call. These set of changes are available in the VEBA Github examples [repository](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/python).

A templatized version of this updated function is available [here](https://github.com/pksrc/vebafn/tree/master/python/md-python-fn) for you to get started that you can get as shown below

``` {.jr .js .jt .ju .jv .ml .mm .mn}
git clone git@github.com:pksrc/vebafn.git vebafn
cd vebafn/python/md-python-fn
├── README.MD
├── pdconfig.json          > provides any configuration for the fn
├── handler
│   ├── handler.py
│   └── requirements.txt > specify any libraries needed for the fn
├── stack.yml
```

For my function to trigger a PagerDuty incident, i needed the following for the API call (more details [here](https://v2.developer.pagerduty.com/docs/send-an-event-events-api-v2))

{{< admonition type=warn title="Note" open=true >}}
API URL — https://events.pagerduty.com/v2/enqueue 
Routing Key — *generated in PagerDuty*
Action — “trigger” 
{{< /admonition >}}

{{<image src="https://miro.medium.com/max/776/1*l6u0P4EmEqYDt-Wm4cJZBQ.png" caption="Stack.yml — Passing the config to the Function" class="imageright">}}

{{< admonition type=warn title="Note" open=true >}}
Routing key and Action are specific to the API call for Pagerduty
{{< /admonition >}}


While the API URL seems static and the other configurable information can be setup to be passed as a config to my function. This is done by using the `secrets` keyword within the stack.yml as shown to the left.

{{<image src="https://miro.medium.com/max/784/1*QULswrrFQMU7ZF-gj6b8dw.png" caption="config.json — configuration for the Function" class="imageright">}}

The config file is a json (can be any other format) with the details about the routing key and the action for the API and is structured as shown to the left.

{{<image src="https://miro.medium.com/max/712/1*deHVUWw4Vo7X062EG8eTCA.png" caption="requirements.txt — library required for the Function" class="imageright">}}

As we need to make http requests, we need the necessary libraries which can be specified in the `requirements.txt` file within the handler folder.

With these files in place, I’m now ready to write my core business logic in the `handler.py` file using the approach below

### Pseudocode
1. Parse the CloudEvent provided as stdin variable to JSON
2. Read the Configuration file - /var/openfaas/secrets/pdconfig
3. Validate the above input data as needed and build the API body 
4. Make the API call to the PagerDuty endpoint

## Start Coding

{{<image src="https://miro.medium.com/max/1282/1*uMazFUuMR4HX2gsdffTmtw.png" width="50%" class="imageright">}}

### **1. Parse the CloudEvent**

This step was already explored in the earlier steps shown above where we parse the raw text to json with exception handling to catch non-json formats

{{<image src="https://miro.medium.com/max/1290/1*kw-o148f0K-f1yX1gEha-A.png" width="50%" class="imageright">}}

### **2. Read the Configuration file**

The config is made available to the function at the location `/var/openfaas/secrets/<configname>` within your container

Here we are reading the Config file that is made available to our function under the location. I’m parsing this file and also validating that the file has the config that are required for this function to work successfully.

{{<image src="https://miro.medium.com/max/1278/1*xf68yICykwwppPO5XITdAQ.png" width="50%" class="imageright">}} 

### **3. Validate Event and Config + Build API Body**

In this section, we are attempting to build the API body as well as validating if the Event has all the keys that my function needs. If the keys are not available (because the fn gets an different event than what it was coded for), then we handle the exception

### **4. Make the API call**

{{<image src="https://miro.medium.com/max/1280/1*NPDmrP5whr5wR-42ipf1Ug.png" width="50%" class="imageright">}}

Finally, we are ready to make the API call with the URL and the body that we have generated. We are also catching any unhandled exceptions that may occur in this process. Don’t forget to close the session.

{{< admonition type=info title="Info" open=true >}}
There are a few helper classes which help with the readability of the code that i’ve utilized in this function but not required.
{{< /admonition >}}

## Wrap Up
**Thats it!** We now have a FaaS fn ready for testing and deployment. You can utilize the unit tests that I’ve provided in the `handler.py` or alternatively deploy the function using the `README.md` file to OpenFaaS hosted on VEBA.

Let me know if you’ve found this helpful. In the next post, i’ll provide the next iteration of changes that will help our function be more clean, debuggable and robust with the help of some util functions and best practices.

Happy Eventing!
