---
title: "Securing Outlook from Unmanaged Devices with Tunnel"
date: 2021-07-06T00:00:00-00:00
draft: false
categories: [EUC]
author: Kevin Ten Eyck
authorlink: https://www.linkedin.com/in/kteneyck/
tags:
- Tunnel
- Office365
- ConditionalAccess 
- WorkspaceONE
lightgallery: true
---

Howdy! I'm Kevin Ten Eyck, a Staff EUC Customer Success Architect at VMware. I've been with VMware for 4 years and greatly enjoy all things EUC! Coming from a background in Mobility and VDI, I love finding new challenges to be solved. Would love to hear your feedback or alternate ideas on how you've solved similar problems!  

## What problem does this solve? 
This blog post is targeted at Workspace ONE UEM administrators who are being asked to prevent unmanaged devices from accessing Office 365 applications. Typically some of the biggest concerns center around, but aren't necessarily limited to: 
- Access to the Outlook for iOS or Android application. 
- Access to applications like Teams or OneDrive. 

To be fair, the solution I am proposing is one of several methods. In fact, VMware and Microsoft have collaborated on an effort to allow Workspace ONE UEM device compliance status to be used to evaluate conditional access policies on Azure application. This is done by essentially daisy chaining the trust and device ID from the Authenticator app on a mobile device and using Workspace ONE Intelligence to communicate that status to Azure. It works fantastic, especially if you are already leveraging Authenticator as your MFA provider. However, it's not perfect and can be difficult to get rolled out. I've provided some documentation below, in case you want to read up on the details.
- [Use Compliance data in Azure AD Conditional Access policies by integrating Workspace ONE UEM with Microsoft](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/services/Directory_Service_Integration/GUID-800FB831-AA66-4094-8F5A-FA5899A3C70C.html)
- [Create a Device-based Conditional Access Policy](https://docs.microsoft.com/en-us/mem/intune/protect/create-conditional-access-intune)
- [Workspace One UEM 3rd party compliance integration – Microsoft Graph API](https://digitalworkspace.one/2020/05/12/compliance-api-msft/)

## Partner Compliance Conditional Access Option
Some of the largest challenges around the above option are really human challenges. Namely:
- Requiring your users to install the Microsoft Authenticator App. This can be challenging if your user already has the app installed, or if they've installed it outside of an Android Work Profile, for instance, where your managed apps will be deployed.
- Requiring your users to manually register their device in Azure AD, beyond just installing the app and signing in for MFA challenges.
- Requiring your users to tap on a web link you push to their device via the Hub Catalog or App Catalog, or try to sign in to a Conditional Access protected app and failing, which provides instructions on how to trigger VMware Intelligent Hub to "trust" the Microsoft Authenticator app. It's fraught with peril due to human error and requires your end-users to do something before they can be granted access.
- Sometimes the device can take a while to populate in Azure's devices portal, which can cause false denials. We push compliance status in real time to Azure, but it can take a bit to reflect.
- The conditional access policies in Azure are tied to some pretty broad categories, so you can require conditional access to all Office 365 apps, but couldn't explicitly allow unmanaged access to something like Microsoft Word, for instance. It's a bit of an all-or-nothing approach. You can get somewhat granular by selecting "Exchange Online" to prevent access just to Outlook app, but that also affects things like Teams.

## Using Tunnel and UAG!

**"Well that all sounds difficult"**, you may be thinking. What can we do instead? I'm glad you asked! There is a really simple and quick way to leverage the device trust you already have with your enrolled devices, and get more value out of the investment you've already made in Workspace ONE Tunnel and the VMware Unified Access Gateways deployed.

The TL;DR of this article is this: We'll send our traffic through our UAGs, then tell Azure Conditional Access "Devices coming from this IP are A-OK, let 'em through!" while defaulting for all others (like an unmanaged mobile device, for example) to being denied access, or subject to some other conditional access policy if you prefer.

The nice thing is if you're already leveraging Tunnel and Per-App VPN then there aren't too many complicated steps needed!

## Bandwidth Concerns and DTRs

**"But Kevin, sending all my email (or whatever O365 app) traffic through my UAGs sounds difficult and painful!"**
Fear not, reader! The good news is we can leverage not only Per-App VPN to only send traffic in the apps that we bless, but also specify the URLs that we want to send traffic through. Like, say for example...our O365 Login page only. This means that the Tunnel will kick in when the user on a managed devices launches an O365 app and tries to login, but the traffic after the login will process through their normal connection (outside the Tunnel).

Let's get into it! 

## Setup

**Pre-requisite**: If you haven't already deployed a Unified Access Gateway with Tunnel setup on Workspace ONE UEM, please read the following TechZone Articles and proceed after you have UAGs deployed with a public IP address and you can communicate with WS1 UEM successfully: 

- [1. Deploying VMware Unified Access Gateway](https://techzone.vmware.com/deploying-vmware-unified-access-gateway-workspace-one-operational-tutorial).
- [2. Configure VMware Tunnel Service](https://techzone.vmware.com/configuring-vmware-tunnel-edge-service-workspace-one-operational-tutorial).
- [3. How to Deploy and Configure Workspace ONE Tunnel App](https://techzone.vmware.com/deploying-vmware-workspace-one-tunnel-workspace-one-operational-tutorial)


{{<image src="/img/euc/tunnel-o365-conditional-access/tunnel-api-success.png" caption="What your connection test should look like when your UAG can talk to UEM">}}

### Step 1: Create Device Traffic Rules 
Remember that we only are interested in tunneling the traffic for the authentication into your O365 apps, not the app traffic itself. How we achieve this is by leveraging Device traffic Rules (DTRs). Here are the URLs that I've supplied in my configuration. I also recommend adding something like ipchicken.com or whatismyip.com so you can validate from a web browser on a managed device that the traffic rules are applying correctly. (You should see the UAGs IP Address when visiting ipchicken rather than your Home IP address or Cellular IP address).

- To create or edit a device traffic rule you'll go to Groups & Settings -> Configurations -> Tunnel.
{{<image src="/img/euc/tunnel-o365-conditional-access/dtr-edit.png" caption="Click the Edit button to modify your rules">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/dtr-edit2.png" caption="Choose your Per Application Assignment to edit by clicking on the Assignment Name">}}


You'll want to create a new rule if you don't already have any traffic tunneled. Over on the left side you'll want to select what applications should traverse the Tunnel when a matched URL is found. This is how we can apply the conditional access only to apps that we bless from UEM. For example, in my screenshot below I have listed Teams and Outlook, but not Word or Powerpoint.
{{<image src="/img/euc/tunnel-o365-conditional-access/dtr.png" caption="Device Traffic Rules for O365">}}

- Your URL list in the freeform text box on the right should be a single line, separated by a comma and no spaces. Note that you cannot choose "*" as an endpoint, but you can use the wildcard to allow for subdomains.
- In my example above, I've listed login.microsoftonline.com,*.windows.net,ipchicken.com

**Tip from the field**: If you don't see your app of choice available to select in the dropdown on the left of your "TUNNEL" traffic rule, you may not have applied an SDK Policy, or have not allowed Tunnel in your SDK settings. Double check your apps to be certain!
{{<image src="/img/euc/tunnel-o365-conditional-access/sdk-list.png" caption="What the list of apps available looks like">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/sdk.png" caption="Check that your app has a SDK Profile selected">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/enablesecurity.png" caption="Check that your App Security settings allow for Tunnel and your DTRs are selected">}}

### Step 2: Create A Trusted Location in Azure for your UAG Public IPs.
We'll want to login as an Azure Admin and navigate to Conditional Access, then Named Locations.
{{<image src="/img/euc/tunnel-o365-conditional-access/trusted-location1.png" caption="Click 'IP Ranges Location' and populate it with your public IPs of your UAG appliances.">}}
- You'll define single IP addresses with CIDR notation. For a single IP address you can provide it in the format of 1.2.3.4/32. If you need help with translating CIDR or creating a CIDR mask that will cover your UAG IPs, I recommend [This Tool](https://www.ipaddressguide.com/cidr)
{{<image src="/img/euc/tunnel-o365-conditional-access/trusted-location2.png" caption="Be sure to check the box for 'Trusted Location'">}}

### Step 3: Create an Azure Conditional Access Policy
In this step, we will be creating the conditional access policy (in this instance, for iOS).
-While still in the Azure Admin Portal under Conditional Access, click on "Policies" in the leftmost navigation tree.
{{<image src="/img/euc/tunnel-o365-conditional-access/newpolicy.png" caption="Click '+ New Policy' to get started">}} 

- Name your policy. Keep it descriptive and include the platform and/or apps included for future troubleshooting.
- Select the Users and Groups you want affected. This is helpful when pilot testing to make sure you don't affect everyone while testing. (You **are** testing in a UAT environment, right?)
- Select the Cloud apps or actions. In my example, I've selected 'Office 365 Exchange Online' and 'Apple Internet Account' (for iOS Native Mail).
{{<image src="/img/euc/tunnel-o365-conditional-access/cloudapps.png" caption="Cloud Apps Selected">}} 
- Select the Conditions for your policy. In my example, I'm targeted iOS, so for Device Platforms, click 'Not configured' under Device Platforms and choose Configure -> 'Yes' -> Include -> Selected device platform -> Check the box for 'iOS'.
{{<image src="/img/euc/tunnel-o365-conditional-access/platforms.png" caption="Platform Selected">}} 
- While still in the Conditions, click on 'Not configured' under Locations.
- Select Configure -> Yes -> Include -> Choose 'Any location'. ALSO in this same tab, click on 'Exclude' and choose 'Selected Locations' then pick your 'UAG' Location Range defined earlier.
{{<image src="/img/euc/tunnel-o365-conditional-access/locations1.png" caption="Making sure all locations are affected by this policy.">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/locations2.png" caption="Excluding your UAG's public IP addresses from this policy.">}} 
- Under 'Access Controls' select 'Grant' and choose 'Block Access'. This is what happens to clients who are NOT coming from the UAG IP address range.
{{<image src="/img/euc/tunnel-o365-conditional-access/grantdeny.png" caption="This portion defines what should happen when this policy does apply (non UAG IPs).">}} 

This will effectively make this policy apply to all iOS Devices trying to access Exchange Online and require their authentication request to originate from a UAG's public IP address.
### Step 4: Test and Validate on an Unenrolled Device and Enrolled Device.
The last step is to make the policy active. For testing, you'll want to scope this to a test group of users until you know everything is working properly.
{{<image src="/img/euc/tunnel-o365-conditional-access/enable.png" caption="Enabling your policy make some time to be effective. I'd give it 15 minutes or so before you test.">}}

{{< admonition type=failure title="Troubleshooting Tips" open=true >}}

- If a user is being denied that should be granted access, first check that a conflicting Azure Conditional Access Policy isn't applying to the user or device. This logging should be available in the Azure Portal under [Monitoring - Sign-Ins](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns).
- In the sign in logs, you'll see the Username, IP address, Application, User-Agent, Device Info (Browser, iOS Version, etc.). There is also a tab for Conditional Access under each log entry that will tell you which Conditional Access Policies were applied and whether they were successful or not. Under each entry, you can click the '...' icon and select 'Show details...' to get in depth details on what from the policy matched. You get a helpful image that shows a check boxes for each assignment and condition, as well as access controls.
{{<image src="/img/euc/tunnel-o365-conditional-access/troubleshooting1.png" caption="A Sample Sign-In Log Entry">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/troubleshooting1a.png" caption="Conditional Access Policies Matched">}}
{{<image src="/img/euc/tunnel-o365-conditional-access/troubleshooting2.png" caption="Conditional Access Details Expanded">}}

- Also, if the Sign-In logs show Successful, but the user sees an error message indicating that they failed to sign in, there may be a conflicting App Protection Policy from the endpoint management perspective.
{{<image width="50%" src="/img/euc/tunnel-o365-conditional-access/failedtosignin.png" caption="This error typically means there was a conflict or issue from the Intune Endpoint App Protection Policies.">}}

- For validation that you have the Outlook app added correctly to be tunneled, as of Tunnel for iOS 21.04, you now have the ability to see configured applications directly within the Tunnel client. This can be enabled by adding the key-value pair DisplayAppAccessList = true to the app config when pushing the app.
{{<image width="50%" src="/img/euc/tunnel-o365-conditional-access/allowedapps.png" caption="Your end user device will be able to see what apps are allowed to use Tunnel with the KVP above.">}}

{{< /admonition >}}
### Results and Gotchas
- The expected experience is that when an enrolled device launches a managed app that we've added to our Device Traffic Rules, and it matches an Azure Conditional Access Policy - the Tunnel Per-App VPN should kick in, but only during authentication. The managed device will be coming from the Tunnel UAG IP address, and therefore be allowed through without any further authentication needed. If you're leveraging Workspace ONE Access and UEM for Mobile SSO in Azure AD, you could even make the sign in process fairly transparent for your end-users.
{{< youtube SVmkbYc399c >}}

- The expected experience for a non-enrolled device is that when they authenticate, they'll hit the policy that says 'if you're not coming from one of these IPs, then I am supposed to deny you.' -- they'll get a sign in failed error message that lets them know they didn't meet the conditions needed to sign in.
{{< youtube Vkn0YeKNB-M >}}


## Compliance and Azure Token Revoke Action

This is all great - but what happens if a device falls out of compliance? How do you make sure they are logged out of their O365 apps? Compliance engine to the rescue!

- To ensure that devices that are non-compliant or are enterprise wiped lose access to the O365 apps, I highly recommend leveraging our Compliance engine's action of 'Azure Token Revoke'. You can read more about it in our [documentation](https://docs.vmware.com/en/VMware-Workspace-ONE-UEM/services/UEM_Managing_Devices/GUID-AWT-COMPLIANCEPOLICYACTIONSBYPLATFORM.html?hWord=N4IghgNiBc4F4FcBOBTABAFwPYGsUDsQBfIA).
- Please note that this requires 'Use Azure AD For Identity Services' enablement in Settings > System > Enterprise Integration > Directory Services > Advanced. Also important to note that this action affects all devices for a given user, disabling any app that relies upon the Azure token. So they'll need to sign in again on any device leveraging the Azure token.

Check the video below for an overview of this functionality.

{{< youtube Y1bDFZEgVuY >}}

One final gotcha is that some of the O365 mobile apps share authentication tokens, so you'll need to think through what applications you are trying to allow through and which ones you are trying to deny access to. An example is that if you create a conditional access policy for Exchange Online, you'll also be affecting Teams and Sharepoint mobile, as they rely on functionality between apps.
