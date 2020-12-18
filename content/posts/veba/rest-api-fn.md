---
title: "A Function for all REST APIs"
date: 2020-05-07T00:00:00-00:00
lastmod: 2020-12-18T00:00:00-00:00
draft: false
categories: [VEBA]
tags:
- event-driven
- cloud-native
lightgallery: true
---

{{< admonition type=info title="Info" open=true >}}
This post was originally published in Medium - [pkblah.medium.com/a-function-for-all-rest-apis](https://medium.com/a-function-for-all-rest-apis-403b4fdf15e8)
{{< /admonition >}}

and…vCenter integration with Slack, PagerDuty, ServiceNow, Zendesk, JIRA, ServiceDesk is now possible with VMware Event Broker Appliance (VEBA)!

Incident Management Systems — ✅

{{<image src="https://miro.medium.com/max/2266/1*0D91nXWygWVXeppMwggXUQ.png" >}}

If you don’t see how the title and the introduction are connected, read on! Let me explain how I wrote one function that makes seemingly all 3rd party system integration easy and possible!

## The Why?

I wrote my very first [serverless function](https://medium.com/@pkblah/writing-your-first-serverless-function-23508cb4ea11?source=friends_link&sk=90cbed9b0dadb67578cebe54a88df494) that allows you to integrate with [vCenter with PagerDuty](https://medium.com/@pkblah/integrating-vcenter-with-pagerduty-241d3813871c?source=friends_link&sk=7b182fdcc37f02bd930d23f77e56213b) (using REST API) and this gave me a very good understanding of our User’s journey through writing a function. I leveraged this experience to brainstorm ways to improve their experience as well as ease their learning curve when it comes to writing a function!

## The What?

I immediately started going down two possible improvement paths

**The first** — Can I make it easy for our Users to get started with our functions so they don’t have to start writing from scratch? I started thinking about abstracting a few steps away for our Users and wrote this [article](https://medium.com/@pkblah/serverless-function-templates-available-2642bb92f58b) that enables writing functions easier by making utility code, test cases, documentation prefilled and readily available!

{{< admonition type=info title="Serverless function templates" open=true >}}
[Serverless function — Templates available](https://medium.com/@pkblah/serverless-function-templates-available-2642bb92f58b)
{{< /admonition >}}

```
git clone git@github.com:pksrc/vebafn.git vebafn #get started with the basics, minimal code, lots of possibilities :)   
cd vebafn/python/sm-python-fn #utilities available, structure provided to customize  
cd vebafn/python/md-python-fn #for the sophisticated function writers - tests, logging etc  
cd vebafn/python/lg-python-fn
```

**The second** — If our Users wanted to integrate with another system that makes REST API available, must our Users rewrite the same code that I wrote to integrate with PagerDuty? Can I possibly make it easy for them?

## The How?

One function with roughly 200 lines of code and VMware Event Broker Appliance (VEBA)!

{{<image src="https://miro.medium.com/max/2560/1*P2vfCLFkpXBwzLdhOv2VUg.gif" >}}

> Would you believe it if I said that setting up these systems took me longer than getting them to work with VEBA?

Before I dive into the function itself, let’s understand the components for making a REST API call. Typically, you’ll need —

{{<image src="https://miro.medium.com/max/1290/1*mZGBQtc8H1WKLmlMKcC2wA.png" caption="metaconfig-servicenow.json" class="imageright" >}}

*   **URL**: http(s)://system.example.com/v1/rest/object
*   **METHOD:** usually POST but could be PUT, PATCH or DELETE etc
*   **HEADERS**: API specific but usually in some sort of key value pair
*   **AUTHN**: Username and Password or None in some cases
*   **BODY**: API specific but takes some sort of data in json (or xml)

This function aims to take these information needed to make a REST API call as configurational values and provides a way for you to pull information from the events, automatically map them to the body through `mappings` as shown below before firing the API request.

{{<image src="https://miro.medium.com/max/3940/1*75YL71SyZe8R7uYBJhsXog.png" >}}

In this very simple ServiceNow example (screenshot), the value for `short_description` within the `body` will automatically be replaced with the value of the `FullFormattedMessage` within the `data` object that we get from the vCenter Event.

To integrate with more systems, all that you need is to build out this metaconfig for the different systems by providing the Rest API information and the mapping to update the request body with the event information. The function takes care of validating the event, updating the request body with the event information and making a POST request to the API.

> NOTE: Currently this function makes a single POST call to REST APIs to that support basic authentication, token within headers type of authentication or no authentication

That’s how effortless this function along with VMware Event Broker Appliance (VEBA) makes modern integration possible with your SDDC with minimal or at times no coding.

Share your feedback

It would be amazing for folks to utilize this function with other systems that I’ve not explored and share your results in the comments below or let me know in a tweet (@pkblah)!

Happy Eventing!
