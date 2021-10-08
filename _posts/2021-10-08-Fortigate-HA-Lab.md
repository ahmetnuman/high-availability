---
layout: post
title: Fortigate Active Passive HA Lab
tags: Fortigate Active Passive HA Lab
categories: highavailability
---

> Ahmet Numan Aytemiz, 11.08.2021

---

![Image](/img/forti_ha_topology.PNG)

**According to topology i am going to configure:**

- **On the Cisco Nexus Switches**

- enable feature vpc 
- enable feature lacp
- create vlan 10 and vlan 20
- Configure vpc peer keep alive link for vpc domain 1
- Configure vpc peer link for Po1 
- Configure Po1 mode active
- Configure Po1 as a trunk port and allow all vlan
- Configure Po2 mode passive with vpc number 2
- Configure Po2 as a trunk port and allow vlan 10,20
- Configure Po3 mode passive with vpc number 3
- Configure Po3 as a trunk port and allow vlan 10,20
- Configure P04 mode active with vpc number 4
- Configure Po4 as a trunk port and allow vlan 10,20

**On the ACCESS-SWITCH**

- Create vlan 10 and vlan 20
- Create Po1 mode passive 
- Create Po1 as a trunk port and allow vlan 10,20
- Configure e 1/0 as a access port for vlan 10
- Configure e 1/1 as a access port for vlan 20

- **On the Active firewall** 
 - Configure ip address 192.168.222.45/24 for port 1 on the active firewall
 - Configure default route ip address 192.168.222.2 on the active firewall
 - Configure 802.3ad lacp interface and assign port2 and port3 ports in it and name it LACP_LAN
 - Confiure VLAN 10 interface on the LACP_LAN interface and assign 172.31.10.254/24 and name it VLAN_10
 - Configure VLAN 20 interface on the LACP_LAN interface and assign it 172.31.20.254/24 and name it VLAN_20
 - Configure firewall policy from VLAN_10 to port 1 , allow http https icmp for 172.31.10.100 source ip and enable source nat
 - Configure firewall policy from VLAN_20 to port 1 , allow http https icmp for 172.31.20.100 source ip and enable source nat
 - For Active Passive HA Configuration from the gui
   - Select mode active passive  
   - Device priority : 200
   - group name : forti_ha
   - password: password
   - Session pickup : enable
   - Heartbeat interface : Port 4
   - from the cli : configure override enable


- **On the passive firewall config ha from the cli**
   - Select mode active passive  
   - Device priority : 100
   - group name : forti_ha
   - password: password
   - Session pickup : enable
   - Heartbeat interface : Port 4
   - configure override enable


---

## NXOS Configuration 

#### VPC Configuration

```
nxos(config)# feature vpc
nxos(config)# feature lacp
nxos(config)# interface ethernet 1/3
nxos(config-if)# no switchport 
nxos(config-if)# ip address 1.1.1.1/30
nxos(config-if)# no shu
nxos(config-if)# exit
nxos(config)# vpc domain 1
nxos(config-vpc-domain)# peer-keepalive destination 1.1.1.2 source 1.1.1.1 vrf default 
nxos(config-vpc-domain)# exit
nxos(config)# interface ethernet 1/1 , ethernet 1/2
nxos(config-if-range)# switchport 
nxos(config-if-range)# channel-group 1 mode active 
nxos(config-if-range)# exit
nxos(config)# interface port-channel 1
nxos(config-if)# switchport 
nxos(config-if)# switchport mode trunk 
nxos(config-if)# switchport trunk allowed vlan all
nxos(config-if)# vpc peer-link 
nxos(config-if)# spanning-tree port type network
nxos(config-if)# exit
```

```
nxos(config)# show vpc brief
```

#### Create VLAN 10 and 20

```
nxos(config)# vlan 10
nxos(config-vlan)# exit
nxos(config)# vlan 20
nxos(config-vlan)# exit
```

#### Po2 Configuration 

```
nxos(config)# interface ethernet 1/6
nxos(config-if)# switchport 
nxos(config-if)# channel-group 2 mode passive 
nxos(config-if)# no shutdown 
nxos(config-if)# exit
nxos(config)# interface port-channel 2
nxos(config-if)# switchport   
nxos(config-if)# switchport mode trunk 
nxos(config-if)# switchport trunk allowed vlan 10,20
nxos(config-if)# vpc 2
```

#### Po3 Configuration 

```
nxos(config)# interface ethernet 1/7
nxos(config-if)# switchport 
nxos(config-if)# channel-group 3 mode passive 
nxos(config-if)# interface port-channel 3
nxos(config-if)# switchport 
nxos(config-if)# switchport mode trunk 
nxos(config-if)# switchport trunk allowed vlan 10,20
nxos(config-if)# vpc 3
```

#### Po4 Configuration 

```
nxos(config)# interface ethernet 1/4 , ethernet 1/5
nxos(config-if-range)# switchport 
nxos(config-if-range)# channel-group 4 mode passive 
nxos(config-if-range)# no shutdown 
nxos(config-if-range)# interface port-channel 4 
nxos(config-if)# switchport mode trunk 
nxos(config-if)# switchport trunk allowed vlan 10,20
nxos(config-if)# vpc 4
```

---


## NXOS2 Configuration

#### VPC Configuration

```
nxos2(config)# feature vpc
nxos2(config)# feature lacp
nxos2(config)# interface ethernet 1/3
nxos2(config-if)# no switchport 
nxos2(config-if)# ip address 1.1.1.2/30
nxos2(config-if)# no shutdown 
nxos2(config-if)# exit
nxos2(config)# vpc domain 1
nxos2(config-vpc-domain)# peer-keepalive destination 1.1.1.1 source 1.1.1.2 vrf default 
nxos2(config-vpc-domain)# exit
nxos2(config-if-range)# channel-group 1 mode active 
nxos2(config-if-range)# exit
nxos2(config)# interface port-channel 1
nxos2(config-if)# switchport 
nxos2(config-if)# switchport mode trunk 
nxos2(config-if)# switchport trunk allowed vlan all
nxos2(config-if)# vpc peer-link     
nxos2(config-if)# spanning-tree port type network
nxos2(config-if)# exit 

```

```
nxos2(config)# show vpc brief
```

#### Create VLAN 10 and 20

```
nxos2(config)# vlan 10
nxos2(config-vlan)# exit
nxos2(config)# vlan 20
nxos2(config-vlan)# exit
```

#### Po2 Configuration 


```
nxos2(config)# interface ethernet 1/6
nxos2(config-if)# switchport 
nxos2(config-if)# channel-group 2 mode passive 
nxos2(config-if)# exit
nxos2(config)# interface port-channel 2
nxos2(config-if)# switchport 
nxos2(config-if)# switchport mode trunk 
nxos2(config-if)# switchport trunk allowed vlan 10,20
nxos2(config-if)# vpc 2
nxos2(config-if)# exit
```

#### Po3 Configuration 

```
nxos2(config)# interface ethernet 1/7
nxos2(config-if)# switchport 
nxos2(config-if)# channel-group 3 mode passive 
nxos2(config-if)# interface port-channel 3
nxos2(config-if)# switchport    
nxos2(config-if)# switchport mode trunk 
nxos2(config-if)# switchport trunk allowed vlan 10,20
nxos2(config-if)# vpc 3
nxos2(config-if)# exit
```

#### Po4 Configuration 

```
nxos2(config)# interface ethernet 1/4, ethernet 1/5
nxos2(config-if-range)# switchport 
nxos2(config-if-range)# channel-group 4 mode passive 
nxos2(config-if-range)# no shutdown 
nxos2(config-if-range)# interface port-channel 4
nxos2(config-if)# switchport 
nxos2(config-if)# switchport mode trunk 
nxos2(config-if)# switchport trunk allowed vlan 10,20
nxos2(config-if)# vpc 4
```

---

## ACCESS-SWITCH Configuration

```
Switch(config)#vlan 10
Switch(config-vlan)#exit
Switch(config)#vlan 20
Switch(config-vlan)#exit
Switch(config)#interface range ethernet 0/0-3
Switch(config-if-range)#switchport 
Switch(config-if-range)#channel-group 1 mode passive 
Switch(config)#interface Port-channel 1
Switch(config-if)#switchport
Switch(config-if)#switchport trunk encapsulation dot1q
Switch(config-if)#switchport mode trunk
Switch(config-if)#switchport trunk allowed vlan 10,20
Switch(config)#interface ethernet 1/0
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config)#interface ethernet 1/1
Switch(config-if)#switchport access vlan 20

```

## Active Firewall Configuration

#### Port 1 Configuration

```
FortiGate-VM64-KVM # config system interface 
FortiGate-VM64-KVM (interface) # edit port1 
FortiGate-VM64-KVM (port1) # set mode static 
FortiGate-VM64-KVM (port1) # set ip 192.168.222.45/24
FortiGate-VM64-KVM (port1) # set allowaccess https http ssh ping 
FortiGate-VM64-KVM (port1) # end
```

#### Default Route Configuration 

```
FortiGate-VM64-KVM # config router static 
FortiGate-VM64-KVM (static) # edit 1
FortiGate-VM64-KVM (1) # set gateway 192.168.222.2
FortiGate-VM64-KVM (1) # set device port1 
FortiGate-VM64-KVM (1) # end
```

#### LACP Interface Configuration

- **Network > Interfaces > Create New > Interaface**
  - Name : LAN_LACP
  - Type : 802.3ad Aggregate
  - Interafce Members : port2, port 3

![Image](/img/lan_lacp.PNG)

#### VLAN 10 Interface Configuration

- **Network > Interfaces > Create New > Interface**
  - Name : VLAN_10
  - Type : VLAN
  - Interface : LAN_LACP
  - IP/Netmask : 172.31.10.254/24

![Image](/img/vlan_10.PNG)

#### VLAN 20 Interface Configuration

- **Network > Interfaces > Create New > Interface**
  - Name : VLAN_20
  - Type : VLAN
  - Interface : LAN_LACP
  - IP/Netmask : 172.31.20.254/24

![Image](/img/vlan_20.PNG)

#### Firewall Policies

![Image](/img/policies.PNG)

#### HA Configuration

![Image](/img/ha_active.PNG)

**override enable from the cli**

```
FortiGate-VM64-KVM # config system ha 
FortiGate-VM64-KVM (ha) # set override enable 
FortiGate-VM64-KVM (ha) # end

```

----

## Passive Firewall

```
FortiGate-VM64-KVM # config system ha
FortiGate-VM64-KVM (ha) # set group-name forti_ha
FortiGate-VM64-KVM (ha) # set mode a-p 
FortiGate-VM64-KVM (ha) # set password password
FortiGate-VM64-KVM (ha) # set hbdev port4 0
FortiGate-VM64-KVM (ha) # set session-pickup enable 
FortiGate-VM64-KVM (ha) # set override enable 
FortiGate-VM64-KVM (ha) # set priority 100

```

![Image](/img/ha_status1.PNG)

**At the end of the lab i am succesfully pinging from both pc to the internet**

![Image](/img/end_lab.PNG)
