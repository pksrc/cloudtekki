---
title: "2021: What's in my homelab?"
date: 2021-02-15T00:00:00-00:00
draft: false
categories: [SDDC]
tags:
- homelab
- vExpert
lightgallery: true
---


I've talked about why everyone needs a homelab in my [medium article](https://pkblah.medium.com/why-a-homelab-9f604752c623) but never had the courage to submit my homelab to the community [homelab list](https://www.virtuallyghetto.com/home-lab) maintained by William Lam {{% twitter profile="lamw" %}}. Mainly because I wasn't sure how my lab would stack up against everyone elses' submission. With my latest post covering my experience [enabling Tanzu with vSphere](https://cloudtekki.com/post/turning-on-wcp/) in my homelab, I feel like I've gone through the checklist of best practices and I'm proud of my minimalist(if i can say so myself) homelab setup! 

{{<image src="https://media.giphy.com/media/3o6gbdUCgjRauJ3vLG/source.gif" caption="Self Congrats!" width="50%">}}

## Lab BOM

In this post, I'm going to provide an overview of what I've installed, cost of ownership, cost of operations and use cases

### Hardware
- **1x 15U Open Frame rack** - approx $200 on ebay
- **2x Dell R620** - approx $350(for RAM) + $100(for CPUs) + $400(for servers)
  - (unsupported CPU for vSphere 7.0)
  - CPU: 2x E5-2630 v2 
  - Memory: 128GB RAM
  - Storage: 
    - Model: PERC H310 Mini (for monolithic)
    - 2x 500GB HDD
- **2x Dell R730** - approx $815(for RAM) + $100(for SSDs) + $950(for servers)
  - CPU: 2x E5-2620 v3 
  - Memory: 128GB RAM
  - Storage: 
    - Model: PERC H730 Mini
    - 2x 500GB HDD
    - 2x 100GB SSD
- **1x Dell R720XD** - approx $750
  - CPU: 2x Intel® Xeon® E5-2670
  - Memory: 92GB RAM
  - Storage: 
    - Model: PERC H710
    - 12x 3.5" 3TB HDD
    -  2x 2.5" 1TB SSD
- Cables, UPS and Accessories - Under $500

### Networking
  - TP-Link (an old leftover router) 
    - routes to my ISP through a firewall
  - Cisco 2960 Switch (mainly for VLANs)
    - 10.10.10.0/24 - (VLAN 10) Management 
    - 10.10.20.0/24 - (VLAN 20) VM
    - 10.10.30.0/24 - (VLAN 30) vMotion 
    - 10.10.50.0/24 - (VLAN 50) vSAN 
    - 10.10.60.0/24 - (VLAN 60) NSX Overlay/TEP
    - 10.10.70.0/24 - (VLAN 70) NSX Uplink 
    - default route back to TP-Link

### Total Cost
- Hardware: approx $4000
- Electricity: under 50$ pm



## SDDC Setup
  - Mgmt: ESXi 6.7 on 2x R620
    - vCenter 7.0 U1 GA
    - 2x NSX Manager
    - 2x NSX Edge
    - 1x VSAN Witness
  - Compute: ESXi 7.0 GA on 2x R730
    - 2 node vSAN Cluster enabled
    - NSX Overlay Network
    - Tanzu with vSphere Enabled
  - Networking (Load Balancing, Firewall etc)
    - 1x AVI Controller
    - 1x AVI Service Engine
  
## Workloads

### Common Services
- 2x Windows 2019 Domain Controllers + DNS
- 1x Windows 2019 Root CA (configured and shutdown)
- 1x Windows 2019 Subordinate CA
- 1x Windows 2019 Microsoft SQL 
- 1x Splunk (deprecated - poorly configured) - currently utilizing the Syslog capability from Unraid
- 1x Ubuntu Docker (deprecated) - utilizing the Docker functionality from Unraid

### EUC
- WS1 UEM
  - 1x Console/Device Services
  - 1x Unified Access Gateway Appliance
  - 1x Access Appliance
  - 1x Access Connector Appliance
  - Bunch of Windows 10 VMs

- Horizon
  - 1x Connection Server
  - 1x Composer
  - 1x Master Image

- Templates
  - Bunch of Windows 10 Templates (built using Packer)
  - Bunch of Windows Server Templates
  - Bunch of Ubuntu Server Templates

### Kubernetes
- 1x Vmware Event Broker Appliance w/ [OpenFaaS](https://www.openfaas.com/)
  - [VEBA Functions](https://vmweventbroker.io/examples)
- 1x Ubuntu 20.04 Kubernetes Control Node
- 2x Ubuntu 20.04 Kubernetes Compute Node



## Use cases
- Replicate and test Customer Scenarios
- Build Demo environments for Events, Customer Workshops
- Test new Technology and Products
- Learn and test new automation capabilities
  - example - Packer and Terraform



## How it started,How it's going

{{<image src="/img/homelab/ghetto-rack.jpeg" caption="They said I needed a rack for my servers" width="50%">}}

{{<image src="/img/homelab/first-esxi.jpeg" caption="First ESXi Install" width="50%">}}

{{<image src="/img/homelab/first-cpu-swap.jpeg" caption="Upgrading my first CPU" width="50%">}}

{{<image src="/img/homelab/baby-trouble.jpeg" caption="Toddler trouble with the lab" width="50%">}}

{{<image src="/img/homelab/cable-mgmt.jpeg" caption="When I asked my wife to assist with the cable management" width="50%">}}

{{<image src="/img/homelab/baby-proof.jpeg" caption="Lab is now baby proofed" width="50%">}}

## Wrap Up

Hopefully this gives you an idea of the costs involved and helps guide you in the decision making process - starting from procurement to install to configuration. While maintaining a homelab is a expensive ordeal and a huge undertaking in of itself, it gives me the opportunity to continuously learn as well as share those learnings broadly and widely! 

> “Life doesn't put a limit on how much you can learn, you do.” 
  ― Matshona Dhliwayo 

I want to sign off by encouraging anyone that are on the edge to get started today. Please feel free to reach out with any questions or assistance and I along with the #vExpert community will be happy to assist. 

