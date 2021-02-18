---
title: "2020: What's in my homelab?"
hidden: true
date: 2020-06-30T00:00:00-00:00
draft: true
categories: [SDDC]
tags:
- homelab
- vExpert
lightgallery: true
featured: false
---

## Lab BOM
Here is my BOM when I started this Journey to turn on the Workload Management in vSphere 7.0 - I spent a lot on Compute and minimally on storage and networking equipments. 

### Hosts
- **2x Dell R620** (unsupported CPU for vSphere 7.0)
  - CPU: 2x E5-2630 v2 
  - Memory: 128GB RAM
  - Storage: 
    - Model: PERC H310 Mini (for monolithic)
    - 2x 500GB HDD
- **2x Dell R730**
  - CPU: 2x E5-2620 v3 
  - Memory: 128GB RAM
  - Storage: 
    - Model: PERC H730 Mini
    - 2x 500GB HDD
    - 2x 100GB SSD

### SDDC Setup
  - vCenter 7.0 GA
  - Mgmt: ESXi 6.7 on 2x R620
  - Compute: ESXi 7.0 GA on 2x R730
    - 2 node vSAN Cluster enabled (after purchasing a couple of 100GB SSDs)

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

## Evolution

{{<image src="/img/homelab/ghetto-rack.jpeg" caption="They said I needed a rack for my servers" width="50%">}}

{{<image src="/img/homelab/first-esxi.jpeg" caption="How it started" width="50%">}}

{{<image src="/img/homelab/first-cpu-swap.jpeg" caption="Upgrading my first CPU" width="50%">}}

{{<image src="/img/homelab/baby-trouble.jpeg" caption="Toddler trouble with the lab" width="50%">}}

{{<image src="/img/homelab/cable-mgmt.jpeg" caption="When I asked my wife to assist with the cable management" width="50%">}}


