---
title: "Serverless function — Quickstart Templates"
date: 2020-04-29T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
---

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/serverless-function-templates](https://medium.com/serverless-function-templates-available-2642bb92f58b)
{{< /admonition >}}

You have probably have seen my other article that helps you get started writing your very first event-driven function using [VMware Event Broker Appliance](https://github.com/vmware-samples/vcenter-event-broker-appliance) (VEBA) for your vCenter infrastructure. I highlight the steps taken to write my first serverless function and provide templates to help you get started quickly!

> **Writing your first Serverless Function** — [here](https://medium.com/@pkblah/writing-your-first-serverless-function-23508cb4ea11)

I also wanted to build on top of my t-shirt sized templates and provide a template where users can focus on writing their core business logic, be able to test quicker and debug easily!

## Small function - Beginner
**Small** — provides a minimal template to get started from scratch complete with a readymade README.md file

```
git clone git@github.com:pksrc/vebafn.git vebafn  
cd vebafn/python/sm-python-fn
├── README.md      > A descriptive documentation for your fn  
├── handler        > Folder for your scripts  
│   └── handler.py > Python file for your core fn logic  
└── stack.yml      > Descriptor file for OpenFaaS deployment
```

## Medium function - Intermediate
**Medium** — **Small** + work with configurations and ready-made handle function; jump right into writing your business logic.

```bash
git clone git@github.com:pksrc/vebafn.git vebafn  
cd vebafn/python/md-python-fn
├── README.MD  
├── config.json          > provides any configuration for the fn  
├── handler  
│   ├── handler.py  
│   └── requirements.txt > specify any libraries needed for the fn  
└── stack.yml
```

## Large function - Expert
**Large — Medium** + improvements :)

Here are some of the things that I wanted to do

**_Improvements_**

*   _Move helper functions to another file; only have the necessary lines of code within the handler.py_
*   _Make the function debuggable without having anyone touch and rebuild the code_
*   _Make some settings be driven through environment variables_
*   _Improve unit testing for faster function development_

With this in mind, I now have the below structure created and the template is available [here](https://github.com/pksrc/vebafn/tree/master/python/lg-python-fn) for you to build on top of.

```
git clone [git@github.com](mailto:git@github.com):pksrc/vebafn.git vebafn  
cd vebafn/python/lg-python-fn
├── README.MD  
├── config.json            > provides any configuration for the fn  
├── handler  
│   ├── __init__.py          
│   ├── handler.py         > lean & focussed on business logic  
│   ├── requirements.txt  
│   └── util.py            > util functions moved here       
├── stack.yml              > new env variables added here  
└── test.py                > tests have moved here
```

Let’s explore what this looks like..

{{<image src="https://miro.medium.com/max/3624/1*uIklg_xHYGrlH2IDpHbM9A.png" caption="FaaSResponse class" >}}

Our function had a few utility classes that helped sending the responses back to OpenFaaS, this was moved the `util.py` file to keep our `handler.py` clean.

{{<image src="https://miro.medium.com/max/1674/1*8-CJ3W8cZWPxH5iJ8Sgmlw.png" caption="stack.yml for OpenFaaS" >}}

Logging in functions (with the templates that we are using) to stdout and stderr are both combined by default, this can be changed by setting an environment variable `combine_output` to false as shown in the `stack.yml` file.

{{<image src="https://miro.medium.com/max/1562/1*d-AXjTsNiRSYNhT_lOYDCA.png" caption="WARN if debug is enabled" >}}

{{<image src="https://miro.medium.com/max/3628/1*VHr70sziW6wTFlwW0hBWYA.png" caption="Logger utility class" >}}

This allows us to write stderr debug messages while stdout gets sent back to OpenFaaS as the output. The function WARN users that debug is enabled and sensitive information could be logged into the console.

With this in place, a logger function was added to produce colorized output in the console for readability and enhanced debuggability.

{{<image src="https://miro.medium.com/max/1370/1*55ySJbrNuekG6lBodb5DAA.png" caption="test.py provided for local testing" >}}

The `util.py` is now imported into the `handler.py` and abstracted away for the function developer who can now focus only on building the core business logic for their usecase!

I’ve also moved all the test cases to outside of the package and located within the `test.py` and it’s been improved to run multiple test cases in one go. Make sure to edit this file to update with test cases that are relevant to what you are looking to accomplish.

These changes certainly helped me write, test and debug my functions faster. While OpenFaaS has an easy process to build, deploy and invoke your function with an OpenFaaS gateway in place, the idea behind these changes and providing a local test script is to speed up the development/testing process by a few hours.

## See this template in action!

I’ve used this template and structure to build a function that can [make a POST REST API call to any system](https://github.com/vmware-samples/vcenter-event-broker-appliance/tree/development/examples/python/invoke-rest-api) (think cURL or Postman but for VEBA). This enables our users to deploy a readymade function and only have to provide the right configuration to have it work with various external systems. This function has been tested to work with Slack, Pagerduty and ServiceNow!

Let me know how these templates help you innovate for your SDDC with VEBA and we can look into making templates in other languages.

Happy Eventing!
