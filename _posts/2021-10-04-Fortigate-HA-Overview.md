---
layout: post
title: Fortigate HA Overview
tags: Fortigate HA Overview
categories: highavailability
---

> Ahmet Numan Aytemiz , 11.03.2021

---

## What is a Fortigate HA ?

- High Availability links and syncrhronize two or more devices.
- One fortigate act as primary (also called acitve Fortigate). 
- Other fortigates are called secondary or standby devices.
- Primary fortigate syncrhronizes its configuration to the other devices.
- A heartbeat link between all the appliances is used to detect unresponsive devices.

## Active-Passive HA 

- The primary fortigate is the only fortigate device that actively process traffic.
- Secondary Fortigate devices remain in passive mode, monitoring the status of the primary devices.
- If a problem is detect on the primary fortigate, one of the secondary devices takes over the primary role.
- This event is called HA failover.

## Active-Active HA

- All fortigate devices process traffic.
- The primary distrubutes the sessions to the cluster members.
- If primary fails, a secondary takes the session distribution job.

## Fortigate Clustering Protocol (FGCP)

- A cluster uses FGCP to:
  - Discover other fortigate devices that belong to the same HA group.
  - Elect the primary
  - Synchronize configuration and other data
  - Detect when a fortigate fails.

- FGCP runs only over the heartbeat links.
  - Uses TCP port 703 with the different ethernet types values to discover fortigates
    - 0x8890 - NAT mode
    - 0x8891 - transparent mode
  - Uses tcp port 23 with ethernet type 0x8893 for configuration synchronization

- If the primary Fortigate is rebooted or shutdown , it becomes the secondary fortigate and waits for the traffic failover to the new primary , before it reboots or shutdown.

## HA Requirements

- Two to four identical Fortigate devices
  - Same firmware
  - same hardware model and vm license
  - same fortiguard, forticloud and forticlient licences
  - same hard drive capacity and partitions
  - same operatiaon mode (transparent or NAT)
  - same interafaces on the each fortigate connected to the same broadcast domain
  - DHCP and PPPoE interfaces are supported
  - one link (preferably two or more [up to eight]) between fortigate devices for heartbeat.

## Primary Fortigate Election : Override Disabled

- Election Process
  - Greater "Connected Monitored ports" become primary
  - Greater "HA Uptime" become primary (If the ha uptime of a device is at least five min more than th ha uptimes of the other fortigate devices)
  - Greater "Priority" become primary 
  - Greater "Serial Number" become primary
- Override disabled by default
- force a failover 
  -  `diagnose sys ha reset-uptime`
- check the HA uptime differences 
  - `diagnose sys ha dump-by vcluster`

![topology](/img/view_reset.PNG)

## Primary Fortigate Election : Override Enabled

- Election Process
  - Greater "Connected Monitored ports" become primary
  - Greater "Priority" become primary 
  - Greater "HA Uptime" become primary (If the ha uptime of a device is at least five min more than th ha uptimes of the other fortigate devices)
  - Greater "Serial Number" become primary

- Override enabled command

   ```
   config system ha
   set override enable
   end
   ```
- Force a failover : change the ha priority 

## Primary Fortigate Tasks

- Exchanges heartneat "hello" packets with all the secondary devics
- Synchronizes its routing table, dhcp information and part of its configuration to all the secondary devices
- Can synchronizes the information of some of the traffic sessions for seamless failover
- In active/active mode 
  - distrubute specific traffic among all the devices in the cluster

## Secondary Fortigate Tasks

- Monitors the primary signs of failure using hello or port monitoring
  - If a problem detects wit the primary , the secondary devics elect a new primary
- In active/acitve mode only;
  - Process traffic distrubuted by the primary 

## Heartbeat Intetrface Ip Address

- The cluster assigns virtual ip address to heartbeat interfaces based on the serial number of each fortigate
 - 169.254.0.1 --> for the highest serial number
 - 169.254.0.2 --> for the second highest serial number 
 - 169.254.0.3 --> for the third highest serial number (and so on) 

- Cluster uses these virtual IP addresses to:
  - Distinguish the cluster members
  - Update configuration changes to the cluster members

## Heartbeat Ports and Monitored Ports

- Heartbeat ports contain sensitive cluster configuration information
  - Must have one heartbeat interface , but using two for redundancy is recommended
  - Not use, vlan subinterfaces, IPSEC vpn tunnel interfaces, redundant interfaces, 802.3ad aggregate interface or forti switch port
  - Use phsical interfaces to heartbeat

- Monitored ports are usually netwroks (interfaces) processing high priority traffic
  - Avoid configuring interface monitoring for all interfaces
  - do not monitor dedicated heartbeat interfaces
  - can monitor vlan interface
  - wait until a cluster is up and running and all interfaces are connected before enabling interface monitoring  

## HA Complete Configuration Synchronization

- When you add a new fortigate to the cluster
  - the primary fortigate compares its configuration checksum with the new secondary fortigate configuration checksum
  - ıf the checksum doesnt match, the primary fortigate uploads its complete configuration to the secondary fortigate

## HA Incremental Configuration Synchronization

- Incremental synhronizations also include;
  - Dynamic data such as dhcp leases, routing table updates, IPSEC SAs, session information ...
- Periodically HA checks for synchronization
  - If CRC checksum values match , cluster is in sync
  - If checksum dont match after five attempts, secondary downloads the whole configuration from the primary 

## What is not Synchronized ?

- HA management interface settings
- In-band HA management interface
- HA override
- HA device priority
- HA virtual cluster priority
- Fortigate Hostname
- Ping server ha priorites
- Licences
- Cache (forti guard, web filtering, email filter, web cache and so on)

## Session Synchronization

- We can enable session table synchronization for most tcp and ipsec vpn sessions

  ```
  config system ha
  set session pickup enable
  end
  ```
- We can enable Synchronization for UDP and ICMP sessions

```
config system ha 
set session pickup enable
set session pickup connectionless enable
end
```

- We can enable Synchronization for multicast sessions

```
config system ha 
set multicast-ttl <5-3600 sec>
end
```

- We can not enable sync for SSL VPN sessions

## Failover Types

- Device failover
- Link failover
- Session failover
- Memory utilization failover
- When failover occurs an event log is generated
  - Optionally , fortigate can also generate an SNMP trap an an alert email

## Virtual Mac Addresses and Failover

- On the primary each interface is assigned a virtual mac address
- Upon failover , the newly elected primary adopts the same virtual mac address as the former primary
