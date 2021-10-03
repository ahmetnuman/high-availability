---
layout: post
title: Palo Alto HA Lab
tags: Palo Alto HA Lab
categories: highavailability
---

> Ahmet Numan Aytemiz , 03.10.2021

- **Configure Active Passive HA According to Topology**
- I already preconfigured on PaloAlto1
  - Managment IP address
  - Zones
  - Interface
  - Virtual Router and Default Route
  - Licensed
  - Security Rule and Source NAT 
- I already preconfigure on PaloAlto2
  - Managment IP address
  - Licensed
  - There are no any other configuration on the PaloAlto2

![topology](/img/pa_ha_lab.PNG)


---

- **On the Palo Alto1 Configure Ethernet 1/3 and Ethernet 1/4 as a HA Link**
  - **Network > Interfaces > Ethernet > Ethernet 1/3**

![topology](/img/pa1_ethernet13.PNG)

  - **Network > Interfaces > Ethernet > Ethernet 1/4**

![topology](/img/pa_ethernet14.PNG)

- **On the PaloAlto1 enable HA, configure goup id , select ha mode, enable config sync , and configure the peer ip address**
  - **Device > High Availability > General > Setup**

![topology](/img/ha_setup.PNG)

- **On the PaloAlto1 configure ethernet 1/4 as a control link and assign ip address/netmask**
  - **Device > High Availablity > General > Control Link (HA1)**

![topology](/img/control_link.PNG)

- **On the PaloAlto1 configure ethernet 1/3 as a data link and assign ip address/netmask and enable session sync**
  - **Device > High Availablity > General > Data Link (HA2)**

![topology](/img/data_link.PNG)

- **On the PaloAlto1 configure device priority 99 and preemption enable**
  - **Device > High Availablity > General > Election Settings**

![topology](/img/prio.PNG)

**and finally commit**

---

- **On the Palo Alto2 Configure Ethernet 1/3 and Ethernet 1/4 as a HA Link**
  - **Network > Interfaces > Ethernet > Ethernet 1/3**
  - **Network > Interfaces > Ethernet > Ethernet 1/4**

![topology](/img/pa2_ha.PNG)

- **On the PaloAlto2 enable HA, configure goup id , select ha mode, enable config sync , and configure the peer ip address**
  - **Device > High Availability > General > Setup**

![topology](/img/pa2_setup.PNG)

- **On the PaloAlto2 configure ethernet 1/4 as a control link and assign ip address/netmask**
  - **Device > High Availablity > General > Control Link (HA1)**

![topology](/img/pa2_e14.PNG)

- **On the PaloAlto2 configure ethernet 1/3 as a data link and assign ip address/netmask and enable session sync**
  - **Device > High Availablity > General > Data Link (HA2)**

![topology](/img/pa2_e13.PNG)

- **I will leave as default election setting because of device priotiy is 100, and i want to make second device as a passive firewall, and i will not configure this device as a preemptive**
  - **Device > High Availablity > General > Election Settings**

![topology](/img/elect2.PNG)


**finally commit**
---

## Monitor HA Status

- **On the PaloAlto1 Dashboard > High Availability**

![topology](/img/ha_status.PNG)

## Verify Zones, Interface Config and Security Rules on the PaloAlto2

- Zones : Network >> Zones

![topology](/img/zones2.PNG)

- Interfaces : Network > Interfaces

![topology](/img/interfaces2.PNG)

- Security Rules: Polices > Security

![topology](/img/security2.PNG)


## Manuel Failover Active Router

- On the PaloAlto1 suspend this device to trigger ha failover
  - Device > High Availability > Operational Commands > Suspend local device

![topology](/img/suspend.PNG)

- On the PaloAlto2 Verify local device is active and peer suspended

![topology](/img/suspend2.PNG)

- Dont forget the make functional PaloAlto1

![topology](/img/functional.PNG)




