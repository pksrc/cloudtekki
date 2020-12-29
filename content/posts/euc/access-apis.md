---
title: "Access: The tale of the hidden APIs"
date: 2020-10-24T00:00:00-00:00
draft: false
categories: [ACCESS, EUC]
tags:
- Access
- APIs
- Postman
- Developer Tools
lightgallery: true
featured: true
---

# Evolving Access through APIs

Workspace One Access is VMware's Product Line that helps provide seamless single sign on and conditional access for Applications from End User's client devices. It is built using technologies such as 
- SAML ([RFC 7522](https://tools.ietf.org/html/rfc7522))
- OAuth ([RFC 6749](https://tools.ietf.org/html/rfc6749)) and 
- [Open ID Connect](https://openid.net/connect/)

If you are new to the Product, you can find more details about WorkspaceONE(WS1) Access 
- Product Page - [here](https://www.vmware.com/products/workspace-one/access.html)
- Documentation - [here](https://docs.vmware.com/en/VMware-Workspace-ONE-Access/index.html)

In this post, I'm going to cover a neat trick that will help you find and build on the APIs that are available with WS1 Access. 

## First, where are the supported APIs? 

{{<image src="/img/euc/access-apis/vidm-api.png" height="150px" class="imageleft" caption="API Categories for WS1 Access">}}

As a general pointer, All APIs and developer related content for VMware are available in [code.vmware.com](https://code.vmware.com) with API documentations and code samples for VMware Products. Another newer resource is [developer.vmware.com](https://developer.vmware.com/) (in beta as of 10/2020) which is a slick new developer documentation landing page for all VMware products. 

All the supported APIs for WS1 Access can be found [code.vmware.com/idm](https://code.vmware.com/apis/57/idm) as shown below

Additionally, there are also some samples with WS1 APIs on [code.vmware.com](https://code.vmware.com/samples?categories=Sample&keywords=&tags=Identity%20Manager&groups=&filters=&sort=dateDesc&page=) and one such examples is [this one](https://code.vmware.com/samples/7419/okta-to-vmwaccess?h=Identity%20Manager#code) which assists with migrating Apps from Okta to WS1 Access

## What if you need more? 

Well, why would you need more? One very obvious reason is bulk operations - I've come across multiple requests that seek to do a bulk operations on Access such as 

1. Exporting multiple Apps
2. Importing multiple Apps
3. App assignment

As of writing this post, these capabilities are not built into the WS1 Access UI and not available through the documented and supported APIs.

<br/>

{{< admonition type=warning title="Disclaimer" open=true >}}
As a word of precaution, this is definitely for those advanced users who find that Access severely lacks capabilities permitting bulk action. The blog covers undocumented APIs meaning they are not supported
{{< /admonition >}}

## Developer Tools: The trick to finding the right API

The trick to finding these APIs is to use the Developer Tools built into Chrome or any Modern Browser. The Developer Tools is a multi-faceted tool and the one that we want provides a `Fiddler` like functionality allowing us to see all the HTTP requests and responses that are being processed by the browser. This [article](https://balsamiq.com/support/faqs/browserconsole/) provides all the ways to get to the Developer Console on all the major Browsers. Once you have the Network Pane open on your browser, you'll also want to enable the Persist logs checkbox which will persist all the requests and responses between any redirects.

{{<image src="/img/euc/access-apis/api-persist-logs.png" caption="Developer Tools - Persist logs">}}

On the Browser tab with Developer Tools launched, navigate to WS1 Access and perform the action that you are looking an API for (PRO TIP: you may want to clear the logs before you do this). The action that you performed should result in an HTTP request as shown below with the requests and the responses as shown the images below. This is the undocumented API that we are looking for. 

{{<image src="/img/euc/access-apis/api-request.png" caption="Access Search API - Request">}}

{{<image src="/img/euc/access-apis/api-response.png" caption="Access Search API - Response">}}

## Postman: Putting it into Action

Now that we have out API, it's time to put the API into action. Follow along through the screenshots to build your new API request in [Postman](https://www.postman.com/downloads/) which is a great tool and comes in really handy, especially if you are unfamiliar with coding, to make API requests and further automate them (to an extent). 

You can quickly build the request as I've shown through the screenshots below. You'll need the below (which if you recall can be found in the )
- **API request URL** 
- **HTTP method** (POST, GET, PUT, DELETE etc)
- **Authentication** (Bearer, Basic etc)
- Request **Headers** (with Access, each API have their own specific Headers for Content-Type and Accept)
  - **Content-Type** is the type that you are sending in the request
  - **Accept** is the type that is acceptable in the response
- Request **Query Parameters** (key value pairs)
- Request **Body** (in JSON or XML format)

{{<image src="/img/euc/access-apis/postman-params.png"  caption="Access Search API in Postman - Query Parameters">}}
{{<image src="/img/euc/access-apis/postman-auth.png"  caption="Access Search API in Postman - Authentication">}}
{{<image src="/img/euc/access-apis/postman-headers.png"  caption="Access Search API in Postman - Request Headers">}}
{{<image src="/img/euc/access-apis/postman-body.png"  caption="Access Search API in Postman - Request Body">}}
{{<image src="/img/euc/access-apis/postman-tests.png"  caption="Access Search API in Postman - Testing the Request">}}

### How do you Authenticate?
Now, in the UI you logged in and then performed an action. This login action behind the scenes created some cookies and the subsequent requests from the same browser also sent those cookies to tell the server that the request is from an authenticated user. How do we do that in our Postman API request? 

While there are a few ways, I'm choosing to do this through the pre-request action which makes sure I've a valid access token when making the request. This pre-request script is also going to save the access token as a variable that I can reuse. To enable my login without hard coding the value in the script, I've a few environment variables setup in Postman such as 
- vidm_url (the tenant url)
- vidm_un (the admin username)
- vidm_pw (the admin password)

{{<image src="/img/euc/access-apis/postman-pre-req.png"  caption="Access Search API in Postman - Pre-request">}}

## Access Postman Collection 

To make it easier for you, I've made available my postman collection for Access available below. 

**10/2020** - https://www.getpostman.com/collections/c50b7bb22ab87f03bc1a 
- Login
- App Management (CRUD)
- Category Management (CRUD)
- App to Category mapping

## Examples

Here are a list of examples around Workspace ONe which leverage APIs to build completely new capabilities

### 1: [Identity Manager Migration Tool Fling](https://flings.vmware.com/identity-manager-migration-backup-tool)
[Chris Halstead](https://flings.vmware.com/users/chris-halstead) has built a fling which helps export web apps from one tenant and import them into another or the same tenant. A pretty cool way to help with migration scenarios or to perform quick testing in UAT or DEV environments to mirror your PROD Apps. 

### 2: [WS1 SCIM Adapter](https://flings.vmware.com/workspace-one-uem-scim-adapter)
[Joe Rainone](https://flings.vmware.com/users/engineer-b094536e-7e99-4e74-8afc-a50f28e39eb4) and [Matt Williams](https://flings.vmware.com/users/engineer-94e36c2e-0327-4ba9-a456-ba244dc9040e) authored the WS1 UEM SCIM Adapter which allows UEM to leverage cloud-based identity resources. Explained in much more detail in this [post at virtualprivateer.com](https://blog.virtualprivateer.com/ws1-uem-scim-adapter/)

## Wrap up
While some of these tools maybe outdated or not relevant, it gives you a great idea of how these APIs can be leveraged and hopefully this post empowers you create more solutions beyond what the product makes available. 

Thanks for reading! Please share and add comments if you've found this useful! 