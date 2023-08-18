

# Getting Started 

Welcome to NHT Labs.

!!!info
       Estimated time to complete the labs is as follows:

       - **DIY Foundation** - 60 minutes
       - **Prism Central** - 30 minutes
       - **XRAY** - 60 minutes

## What's New

- Workshop uses for the following software versions:
  - AOS 5.20.3
  - Prism Central pc.2022.4.x

## Agenda

- DIY Foundation
- Deploying Prism Central
- XRAY

## Initial Setup

- Take note of the *Passwords* being used from you RX reservation details
- Log into your virtual desktops (connection info below)
- Login to Global Protect VPN if you have access

## Cluster Assignment

The instructor will inform the attendees their assigned clusters.

!!! note

    If these are Single Node Clusters (SNCs) pay close attention on the networking part. The SNCs are setup and configured differently to the 3 or 4 node clusters


## Environment Details

Nutanix Workshops are intended to be run in the Nutanix Hosted POC environment. Your cluster will be provisioned with all necessary images,networks, and VMs required to complete the exercises.

### Networking

As we are able to provide three/four node clusters and single node clusters in the HPOC environment, we need to describe each sort of cluster separately. The clusters are setup and configured differently.

#### Three/Four node HPOC clusters

Three or four node Hosted POC clusters follow a standard naming convention:

- **Cluster Name** - POC*XYZ*
- **Subnet** - 10.**42**.*XYZ*.0
- **Cluster IP** - 10.**42**.*XYZ*.37

For example:

- **Cluster Name** - POC055
- **Subnet** - 10.42.55.0
- **Cluster IP** - 10.42.55.37 for the VIP of the Cluster

Throughout the Workshop there are multiple instances where you will need to substitute *XYZ* with the correct octet for your subnet, for example:

| IP Address     |   Description |
| -------------- | --------------- |
| 10.42.*XYZ*.37 |  Nutanix Cluster Virtual IP   |
| 10.42.*XYZ*.39 |  **PC** VM IP, Prism Central |
| 10.42.*XYZ*.41  |  **DC** VM IP, NTNXLAB.local Domain Controller   |


Each cluster is configured with 2 VLANs which can be used for VMs:


|Network Name     | Address             | VLAN    | DHCP Scope|
|-----------------| ------------------- |-------- | -----------|
|Primary          | 10.42.*XYZ*.1/25    | 0       | 10.42.*XYZ*.50-10.42.*XYZ*.124|
|Secondary        | 10.42.*XYZ*.129/25  | *XYZ1*  | 10.42.*XYZ*.132-10.42.*XYZ*.253|


### Credentials

!!! note

    The *Cluster Password* is unique to each cluster and will be provided by the leader of the Workshop.

| Credential        | Username                 | Password           |
|------------------ |------------------------- |--------------------|
| Prism Element     | admin                    | *Cluster Password* |
| Prism Central     | admin                    | *Cluster Password* |
| Controller VM     | nutanix                  | *Cluster Password* |
| Prism Central VM  | nutanix                  | *Cluster Password* |

Each cluster has a dedicated domain controller VM, **DC**, responsible for providing AD services for the **NTNXLAB.local** domain. The domain is populated with the following Users and Groups:


| Group            | Username(s)              | Password |
|-----------------| ------------------------- |------------|
| Administrators    | Administrator             | default password | 
| SSP Admins        | adminuser01-adminuser25   | default password | 
| SSP Developers    | devuser01-devuser25       | default password | 
| SSP Consumers     | consumer01-consumer25     | default password |
| SSP Operators     | operator01-operator25     | default password |
| SSP Custom        | custom01-custom25         | default password |
| Bootcamp Users    | user01-user25             | default password |


## Access Instructions

Check your HPOC cluster reservation email for details.