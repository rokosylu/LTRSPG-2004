#

![](images/0.0%20Title%20page.png)

#

# Table of Content

Use GitHub TOC

![](images/) ADD A PIC OF A capture of TOC on GITHUB

<br/>
<br/>

#

# Learning Objectives

Upon completion of this lab, you will be able to:

- Describe network slicing methodologies to partition the network infrastructure to provide differentiated services for different SLAs.
- Deploy On-Demand Next-Hop (ODN) policies with Automated Steering (AS) to enable auto-steering of traffic based on the operator SLAs.
- Use Flex-Algo with On-Demand Next-Hop and Automated Steering to allow auto-steering of traffic into a topology, or path, defined by a service provider's logic.

<br/>

>NOTE:
>This _**lab**_ familiarizes the user with the Cisco SR TE and its configuration.Although the lab design and configuration examples could be used as a reference, it's not a real design, so not all recommended features are used, or enabled optimally. For design related questions please contact your Cisco representative, or a Cisco partner.  

<br/>
<br/>

#

# Introduction

Segment routing for traffic engineering (SR-TE) uses a "policy" to steer traffic through the network. An SR-TE policy path is expressed as a list of segments that specifies the path, called a segment ID (SID) list. Each segment is an end-to-end path from the source to the destination, and instructs the routers in the network to follow the specified path instead of following the shortest path calculated by the IGP. If a packet is steered into an SR-TE policy, the SID list is pushed on the packet by the head-end. The rest of the network executes the instructions embedded in the SID list.

An SR-TE policy is identified as an ordered list (head-end, color, end-point):

- Head-end – Where the SR-TE policy is instantiated
- Color – A numerical value that distinguishes between two or more policies to the same node pairs (Head-end – End point)
- End-point – The destination of the SR-TE policy

Every SR-TE policy has a color value. Every policy between the same node pairs requires a unique color value.

SR-TE is the foundation of network slicing.

#
## Overview

In this lab, you will configure and verify SR-TE slicing in a SR-MPLS network by using the following tools:

- Explicit Path
- Automated Steering with latency, te metric, and affinity constraints.
- ODN (On-Demand NextHop)
- Flex-Algo

#
## Physical Topology Diagram

Drawing of the physical connections in the lab.

![](images/physicalTopology.png)

#
## IGP (IS-IS) Topology Diagram

There are 3 different IGP domains "ACCESS 1", "AGG-CORE" and "ACCESS 2" in the lab. There is no redistribution between areas. BGP-LU is used across the domains for end to end loopback reachability.

![](images/IGPTopology.png)

#
## IP and SID Diagram

The SRGB in the lab is configured as 19000-119000. There are 2 VRFs configured "CUSTOMER-A" and "CUSTOMER-B".

![](images/IPandSID.png)

#
## IGP Metric Diagram

The default IGP metric is 100 throughout the topology.

![](images/IGPMetric.png)

#
## Latency Metric Diagram

The default latency metrics of the links are seen below. Latency is lower on the links going through the inline PCE nodes.

![](images/LatencyMetric.png)

#
## TE Metric Diagram

The default TE metric is 1000 throughout the topology.

![](images/TEMetric.png)

#
# Access the Lab

To start, find the anyconnect icon on the taskbar or home screen.

![](images/14ce9d0a18fb7108.png)

Double click on the icon to open the Cisco AnyConnect Mobility Client.

![](images/325ba1b133700f9e.png)

Connect to your POD. The format will be [https://64.100.9.4/CLPODx](https://64.100.9.4/CLPODx) where X is your POD number.

![](images/cd00f0ae868d297d.png)

Once connected a security warning message will be seen. Click "Connect Anyway".

![](images/46a2b563343ce9ea.png)

Enter the username and password for your pod.

![](images/95dab4b1258cd5cd.png)

Click accept to connect to the lab.

![](images/b79d42209b771423.png)

Once connected you will see this popup window.

![](images/7049bb8d24cac6c3.png)

Now you are ready to start the lab

#
## Management IPs of the lab devices

| Router Hostname | Management IP |
| --- | --- |
| R1 | 10.10.2XX.1 |
| R2 | 10.10.2XX.2 |
| R3 | 10.10.2XX.3 |
| R4 | 10.10.2XX.4 |
| R5 | 10.10.2XX.5 |
| R6 | 10.10.2XX.6 |
| R7 | 10.10.2XX.7 |
| R8 | 10.10.2XX.8 |
| PCE1 | 10.10.2XX.11 |
| PCE2 | 10.10.2XX.12 |
| PCE3 | 10.10.2XX.13 |
| CE1 | 10.10.2XX.21 |
| CE2 | 10.10.2XX.22 |
| CE3 | 10.10.2XX.23 |


>NOTE
>Where XX is your POD number. i.e: POD01 will use 01, POD 13 will use 13Username and Password is cisco/cisco for all devices

<br/>
<br/>

#
# Scenario 1 - Lab Verification of the Underlay and Overlay Fundamentals

All IP addresses, IGP protocol configuration, and basic BGP configuration have been completed in the simulation. The purpose of this is for you to focus on SRTE and other advanced topics. However, let's verify all protocols are functional properly, using this as a quick review of the fundamentals as well.

<br/><br/>

## Task 1.1: Verify ISIS Operation

Log into R3 and verify it has an adjacency relationship with R1, R4, and PCE1 in the ACCESS-1 process and R4, R5, and PCE2 in the AGG-CORE Process. There are two processes since this is an ABR.
<br/>
```
show isis adjacency
```


![](images/1.1_1_isisadjacency.png)

You can verify the other ABRs (R4, R5, R6) have adjacencies in both processes.

<br/>
<br/>

## Task 1.2: Verify Segment Routing Configuration and Operation

IOS-XR has a default SRBG of 16000-23999. In this lab, we are not using the default SRBG but rather have configured one from 19000-119000. Verify this is configured correctly on R3.
<br/>
```
show run segment-routing
```

![](images/shRunSegment.png)

>NOTE:
>When changing the SRGB, a reload of the router may be needed. 
<br/>

Verify the size of the SRGB on R3 is the same as what is configured. It starts at 19000 and has a size of 100001 labels.

```
sh mpls label table label 19000 detail 
```
![](images/1.2_2_shMplsLabTab.png)
<br/><br/>

The table below shows the table of the router hostname and the Prefix-Sid values.

| Router Hostname | Prefix SID |
| --- | --- |
| R1 | 19001 |
| R2 | 19002 |
| R3 | 19003 |
| R4 | 19004 |
| R5 | 19005 |
| R6 | 19006 |
| R7 | 19007 |
| R8 | 19008 |
| PCE1 | 19011 |
| PCE2 | 19012 |
| PCE3 | 19013 |

There are two ways to configure the Prefix-Sid value.

- Absolute Value
- Index Value

In this lab, an Index Value is used. Verify R3 has an index of 3.

```
show run router isis 
```
![](images/shRunRouterIsis.png)
<br/>

Check the ISIS segment-routing label table and verify 19003 is the label allocated for the local loopback0 interface. Recall R3 is an ABR and the interface/label will be in both processes.

```
show isis segment-routing label table   
```
![](images/shIsisSegmentLabelTable.png)


<br/>
<br/>

## Task 1.3: Verify Link Protection (TI-LFA)

Topology-Independent Loop-Free Alternate (TI-LFA) uses segment routing to provide link, node, and Shared Risk Link Groups (SRLG) protection in topologies where other fast reroute techniques cannot provide protection.

TI-LFA is enabled under the interfaces in ISIS. Run the following command to verify the configuration.

```
show run router isis ACCESS-1 interface
```

![](images/shRunRouterIsisAccess.png)
<br/>


Checking the details of the primary and backup path the following commands are run.

>NOTE: The primary path information is in GREEN and the backup path information is in BLUE.

```
show isis ipv4 fast-reroute 1.1.1.1/32 detail
```
![](images/1.3_2_shIsisFast.png)
<br/>

```
show cef 1.1.1.1/32
```
![](images/shCef.png)

The output above shows the regular path sending the traffic to the link between R3-R1 via 171.1.3.0 with a ImpNull label (PHP). The backup path sends to traffic to PCE-1 via 172.3.11.1, and imposes the label of 19001.
<br/><br/>

## Task 1.4: Verify BGL-LS

BGP Link-State (LS) is an address family designed to carry an IGP link-state information through BGP. BGP-LS delivers the network topology information to topology servers, or any other application needing to know the topology for decisions.

In this scenario, BGP-LS is used to deliver IGP information between domains. Below is a topology of the BGP-LS peerings.

![](images/bgpLsPeering.png)
<br/><br/>

Verify PCE-1 has a BGP-LS peering to PCE-2 and is receiving all the routes.
```
show bgp link-state link-state summary
```
![](images/shBgpLinkStateSum.png)
<br/><br/>

Recall BGP-LS carries the IGP information into BGP. Check the IGP information for the link between R7 and R8 on PCE-1.

In the example below, look in the link-state database and search for 172.7.8.1 (the link between R7 and R8). The result will give you two entries. One from R7 and one from R8. Take the whole NRLI and enter into the "show bgp link-state link-state" command to view the details.

```
show bgp link-state link-state | i 172.7.8.1
```
![](images/shBgpLsLs.png)
<br/><br/>

## Task 1.5: Verify PCE Communication

All of the SRTE headend work will be done on R1 and R2 in this lab.
```
show segment-routing traffic-eng pcc ipv4 peer
```
![](images/shSegTePcc.png)
<br/><br/>

#
# Scenario 2 - Inter-Domain Network Slicing for Regular Traffic

In this scenario we will get a baseline of the default routing scheme (IGP shortest path) for vrf "CUSTOMER-A" that we can use it to compare with the other scenarios later

## Task 2.1: Verify Service Path

On CE1, issue a traceroute in CUSTOMER-A vrf to loopback 31 interface of CE2 and CE3

```
traceroute vrf CUSTOMER-A 150.22.1.1 probe 1
traceroute vrf CUSTOMER-A 150.23.1.1 probe 1
```
![](images/2.1_traceroute.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |
<br/><br/>
#
# Scenario 3 - Inter-domain SRTE using Explicit-path

In this scenario we will slice our network using explicit path. We will create two explicit paths, depicted below, the green path (color 3222) from CE1 to CE2 Loopback32 (150.22.2.2) and a red path (color 3232) from CE1 to CE3 Loopback32 (150.23.2.2).

![](images/3.0_flows.png)
<br/><br/>

## Task 3.1: Configure color for Explicit-path

We will assign the color for each explicit path at the tail end of each tunnel. The first step is to create an extended community for the colors.

Apply the following configuration in R4:

```
extcommunity-set opaque COLOR-3222
  3222
end-set
```

Since CE3 has two connections to our network using R7 and R8, we will need to assign color 3232 on both routers.

Apply the following configuration in R7 and R8:
```
extcommunity-set opaque COLOR-3232
  3232
end-set
```

After we have created the extended communities, we will need to create the route maps to apply the color to our prefixes, we will apply our route policies in the ingress direction from the CEs.

The route-policy will match our destination IP on the CEs and tag the prefixes with the extended community, then we can use the extended community later to choose the desired path.

Apply the following configuration in R4:
```
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3222 #####
  if destination in (150.22.2.2) then
    set extcommunity color COLOR-3222
  ##### Everything Else #####
  else
    pass
  endif
end-policy
!
router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.4.22.1
   address-family ipv4 unicast
    route-policy CUST-A_SET_COLOR_IN in
   !
  !
 !
!
```

Apply the following configuration in R7 and R8:
```
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path – Color 3232 #####
  if destination in (150.23.2.2) then
    set extcommunity color COLOR-3232
  ##### Everything Else #####
  else
    pass
  endif
end-policy
```

Apply the following configuration in R7:
```
router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.7.23.1
   address-family ipv4 unicast
    route-policy CUST-A_SET_COLOR_IN in
   !
  !
 !
!
```

Apply the following configuration in R8:
```
router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.8.23.1
   address-family ipv4 unicast
    route-policy CUST-A_SET_COLOR_IN in
   !
  !
```

## Task 3.2: Verify the prefixes are tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefixes to make sure they have been tagged with the new color.

In R1 issue the following commands:
```
sh bgp vrf CUSTOMER-A 150.22.2.2
sh bgp vrf CUSTOMER-A 150.23.2.2
```
![](images/3.2_shBgp.png)


>NOTE:
>Output from R2 is similar, omitted for brevity.

<br/><br/>

## Task 3.3: Configure SRTE policy using Explicit-path

To configure an SR-TE policy with an explicit path, we will need to create the segment list and the SR-TE policy.

The segment-list describes each segment on the path that the packets will need to take. It can use and IP address, an mpls label or a combination of both

An Explicit-path policy is identified by the head-end, color and tail-end tuple, in this task we will be configuring at the head-end router the policy with the assigned color and tail-end (end-point).

On R1 & R2 apply the following configuration:
```
segment-routing
 traffic-eng
  segment-list PCE1-R3-R4
   index 10 mpls adjacency 11.11.11.11
   index 20 mpls adjacency 3.3.3.3
   index 30 mpls adjacency 4.4.4.4
  !
  segment-list R3-R4-R6-R5-R7
   index 10 mpls label 19003
   index 20 mpls label 19004
   index 30 mpls label 19006
   index 40 mpls label 19005
   index 50 mpls label 19007
  !
  segment-list R4-R3-R5-R6-R8
   index 10 mpls label 19004
   index 20 mpls label 19003
   index 30 mpls label 19005
   index 40 mpls label 19006
   index 50 mpls label 19008
  !
 
  policy EXP_COLOR-3222_R4
   color 3222 end-point ipv4 4.4.4.4
   candidate-paths
    preference 100
     explicit segment-list PCE1-R3-R4
     !
    !
   !
  !
  policy EXP_COLOR-3232_R7
   color 3232 end-point ipv4 7.7.7.7
   candidate-paths
    preference 100
     explicit segment-list R3-R4-R6-R5-R7

     !
    !
   !      
  !
  policy EXP_COLOR-3232_R8
   color 3232 end-point ipv4 8.8.8.8
   candidate-paths
    preference 100
     explicit segment-list R4-R3-R5-R6-R8
     !
    !
   !
  !
 !
!
```

<br/><br/>
## Task 3.4: Verify Service Path

On R1 & R2 display the BGP prefixes.
```
sh bgp vrf CUSTOMER-A 150.22.2.2
```
![](images/3.4_1.png)
```
sh bgp vrf CUSTOMER-A 150.23.2.2
```
![](images/3.4_2.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 3222
```
![](images/3.4_3.1.png)

```
sh segment-routing traffic-eng policy color 3232
```
![](images/3.4_3.2.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.


On R1 & R2, display cef for CE2 Loopback32 (150.22.2.2) and CE3 Loopback32 (150.23.2.2).
```
sh cef vrf CUSTOMER-A 150.22.2.2
sh cef vrf CUSTOMER-A 150.23.2.2
```
![](images/3.4_4.png)



>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display the binding SID info
```
sh mpls forwarding labels 119008
sh mpls forwarding labels 119011
 ```
![](images/3.4_5_shMplsForLab.png)

>NOTE:
Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

<br/>

>NOTE:
>For the first scenario, we will link everything together from the outputs to verify the label stack. Even though this is only shown here, the same steps can be followed for each scenario.

Before looking at the traceroute let's predict the label stack for traffic hitting the SR-TE policy, entering R1.

For the destination 150.22.2.2, in CEF we see the VPN label is 119013. That will be the last label in the stack.

Also in CEF, we see the binding SID as 119008, and looking the at the SR-TE policy for that B-SID there is an imposed label stack of 19011 / 19003 / 19004.

Since PCE1 (label 19011) is the next hop from R1, it will PHP the top lab and the label stack should be 19003 / 19004 / 119013.

Let's see if that is what we see.

On CE1, traceroute to CE2 Loopback32 (150.22.2.2) & CE3 Loopback32 (150.23.2.2) to display the path taken.
```
traceroute vrf CUSTOMER-A 150.22.2.2 source lo31 probe 1      
traceroute vrf CUSTOMER-A 150.23.2.2 source lo32 probe 1
```
![](images/3.4_6_tracert.png)

>NOTE
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

<br/><br/>
# Scenario 4 - Inter-Domain Network Slicing for Low Latency

In this scenario, we will slice our network using a dynamic path. A dynamic path does not use a segment list, but it calculates on the head end the shortest path to the destination using the metric type selected. In this scenario, we will create a dynamic path from CE1 to CE3 Loopback33 (150.23.3.3) using delay as the metric.

![](images/4.0_flowDiagram.png)

## Task 4.1: Configure color Low Latency Traffic

Similar to what we did for Explicit path, we will assign the color at the tail end of the tunnel.

>NOTE: We can only configure one route-policy to any given neighbor in each direction also inline modification of a route-policy is not allowed, therefore we will need to recreate the route-policy to include previous lines and add our new lines.

Apply the following configuration in R7 and R8:
```
extcommunity-set opaque COLOR-3233
  3233
end-set
!
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3232 #####
  if destination in (150.23.2.2) then
    set extcommunity color COLOR-3232
    ##### Dynamic Path - Latency #####
  elseif destination in (150.23.3.3) then
    set extcommunity color COLOR-3233
    ##### Everything Else #####
  else
    pass
  endif
end-policy
```

>NOTE:
>There is no need to apply the route-policy in the BGP of R7 & R8 towards CE3 since that was done in scenario 2 already.

<br/><br/>


## Task 4.2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the new color.

In R1 & R2 issue the following command:
```
sh bgp vrf CUSTOMER-A 150.23.3.3
```
![](images/4.2_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. 

<br/><br/>


## Task 4.3: Configure SRTE policy Low Latency Traffic

Similar to the explicit path, a dynamic path is identified by the head-end, color and tail-end tuple. In this task, we will be configuring at the head-end router the policy with the assigned color and tail-end (end-point) however instead of using an explicit list, we will be using a dynamic calculation based on the lowest latency path.

On R1 & R2 apply the following configuration:
```
segment-routing
 traffic-eng
  policy DYN-Latency_COLOR-3233_R7
   color 3233 end-point ipv4 7.7.7.7
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type latency
      !
     !
    !
   !
  !
  policy DYN-Latency_COLOR-3233_R8
   color 3233 end-point ipv4 8.8.8.8
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type latency
      !
     !
    !
   !
  !
 !
!
```

>NOTE: For inter-domain SR-TE with multiple IGP without mutual redistribution, it is required to have a PCEP that can collect all the link information from all IGPs, without a PCEP, the head end will not have a complete view of the network and it will be unable to create a valid path that traverses multiple IGP.

<br/><br/>

## Task 4.4: Verify Service Path

On R1 & R2 display the BGP prefix.
```
sh bgp vrf CUSTOMER-A 150.23.3.3
```
![](images/4.4_1_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 3233
```

![](images/4.4_2_shSegTe.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display cef for 150.23.3.3
```
RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.3.3**
```

![](images/4.4_3_shCef.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.


On R1 & R2, display the binding SID info
```
sh mpls forwarding labels 119022
```
![](images/4.4_4_shMplsFor.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On CE1, traceroute to CE3 Loopback33 (150.23.3.3) to display the path taken.
```
traceroute vrf CUSTOMER-A 150.23.3.3 probe 1
```
![](images/4.4_5_tracert.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity.
<br/><br/>

## Task 4.5: Change the Latency value of a link

In this task, we will temporarily make changes to the delay of PCE2-R5 and PCE2-R6 to simulate degradation of those links which will trigger a re-optimization of the path.

On PCE2 increase the delay to R5 and R6 from 10ms to 150ms.
```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 150
  !
 !
 interface GigabitEthernet0/0/0/3
  delay-measurement
   advertise-delay 150
  !
 !
!
```
<br/><br/>

## Task 4.6: Verify Service Path Change

The new SR-TE path will re-optimize to avoid the high delay links.

![](images/4.6_diagram.png)

On R1 & R2 display the SR-TE policy details.
```
sh segment-routing traffic-eng policy color 3233
```
![](images/4.6_1_shSegTe.png)

>NOTE:
>Output from R2 is similar, omitted for brevity.

On CE1, traceroute to CE3 to display the new path taken.
```
traceroute vrf CUSTOMER-A 150.23.3.3 probe 1
```
![](images/4.6_2_tracert.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity.

## Task 4.7: Restore the original Latency value

On PCE2 restore the delay to R5 and R6 from 150ms to 10ms & 13ms respectively.
```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 10
  !
 !
 interface GigabitEthernet0/0/0/3
  delay-measurement
   advertise-delay 13
  !
 !
!
```
<br/><br/>

#

# Scenario 5 - Inter-Domain Network Slicing for TE-Metric

In this scenario, we will slice our network using a dynamic path with TE metric. We will create a dynamic path from CE1 to CE3 Loopback34 (150.23.4.4) using TE metric.

![](images/5.0_diagram.png)

## Task 5.1: Configure color for TE-metric

>NOTE:
>We can only configure one route-policy to any given neighbor in each direction also inline modification of a route-policy is not allowed, therefore we will need to recreate the route-policy to include previous lines and add our new lines.

Apply the following configuration in R7 and R8:
```
extcommunity-set opaque COLOR-3234
  3234
end-set
!
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3232 #####
  if destination in (150.23.2.2) then
    set extcommunity color COLOR-3232
    ##### Dynamic Path - Latency #####
  elseif destination in (150.23.3.3) then
    set extcommunity color COLOR-3233
    ##### Dynamic Path - TE #####
  elseif destination in (150.23.4.4) then
    set extcommunity color COLOR-3234
 
    ##### Everything Else #####
  else
    pass
  endif
end-policy
```

>NOTE:
>There is no need to apply the route-policy in R7 & R8 towards CE3 since that was done in scenario 2 already. 

<br/><br/>

## Task 5.2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following command:
```
sh bgp vrf CUSTOMER-A 150.23.4.4
```
![](images/5.2_1_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. 

<br/><br/>

## Task 5.3: Configure SRTE policy for TE-metric

The configuration of dynamic path with TE metric is similar to the previous scenario where we used latency, the only difference is changing the metric type from latency to TE. Also, since we are going across multiple IGP without mutual redistribution, again we will be using a PCEP to obtain the dynamic path.

On R1 & R2 apply the following configuration:
```
segment-routing
 traffic-eng
  policy DYN-TE_COLOR-3234_R7
   color 3234 end-point ipv4 7.7.7.7
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type te
      !
     !
    !
   !
  !
  policy DYN-TE_COLOR-3234_R8
   color 3234 end-point ipv4 8.8.8.8
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type te
      !
     !
    !
   !
  !
 !
!
```
<br/><br/>

## Task 5.4: Verify Service Path

On R1 & R2 display the BGP prefix.
```
sh bgp vrf CUSTOMER-A 150.23.4.4
```
![](images/5.4_1_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 3234
```
SR-TE policy database
![](images/5.4_2_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 

On R1 & R2, display cef for 150.23.4.4
```
sh cef vrf CUSTOMER-A 150.23.4.4
```
![](images/5.4_3_shCef.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display the binding SID info
```
sh mpls forwarding labels 119026
```
![](images/5.4_4_shMplsForLab.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On CE1, traceroute to CE3 to display the path taken.
```
traceroute vrf CUSTOMER-A 150.23.4.4 probe 1
```
![](images/5.4_5_tracert.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. 

<br/><br/>

## Task 5.5: Change TE metric of a link

In this task, we will temporarily make changes to the TE metric of R1-R3 and R2-R4 links from 1000 to 5000 to force a re-optimization.

On R1 and R2 issue the following command:
```
segment-routing
 traffic-eng
  interface GigabitEthernet0/0/0/3
   metric 5000
  !
 !
!

```
## Task 5.6: Verify Service Path Change

The new SR-TE path should avoid the high TE metric links

![](images/5.6_1_diagram.png)

On R1 & R2 display the SR-TE policy details.
```
RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3234**
```
![](images/5.6_2_shSegTe.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. 

On CE1, traceroute CE3 Loopback 34 to display the new path taken.
```
traceroute vrf CUSTOMER-A 150.23.4.4 probe 1
```
![](images/5.6_3_tracert.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity.

<br/><br/>

## Task 5.7: Restore original TE metric

On R1 and R2 issue the following command:
```
segment-routing
 traffic-eng
  interface GigabitEthernet0/0/0/3
   metric 1000
  !
 !
!
```
<br/><br/>


# Scenario 6 - Inter-Domain Network Slicing with link affinities

Affinity constraints allow the head-end router to compute a dynamic path that includes or excludes links that have specific colors or combinations of colors. We will create a dynamic path from CE1 to CE3 Loopback36 (150.23.6.6) using TE-metric and excluding RED links.

![](images/6.0_digram.png)

<br/><br/>

## Task 6.1: Configure color

For this scenario, we will be using CE3 150.23.6.6 with color 3236, similar to previous scenarios we need to create the extended community and update the route-policy.

Apply the following configuration in R7 and R8:
```
extcommunity-set opaque COLOR-3236
  3236
end-set
!
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3232 #####
  if destination in (150.23.2.2) then
    set extcommunity color COLOR-3232
    ##### Dynamic Path - Latency #####
  elseif destination in (150.23.3.3) then
    set extcommunity color COLOR-3233
    ##### Dynamic Path - TE #####
  elseif destination in (150.23.4.4) then
    set extcommunity color COLOR-3234
    ##### Anycast SID - TE #####
  elseif destination in (150.23.5.5) then
    set extcommunity color COLOR-3235
    ##### Affinity - TE #####
  elseif destination in (150.23.6.6) then
    set extcommunity color COLOR-3236
     ##### Everything Else #####
  else
    pass
  endif
end-policy

```
>NOTE:
>There is no need to apply the route-policy in R7 & R8 towards CE3 since that was done in scenario 2 already. 
<br/><br/>

## Task 6.2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following command:
```
sh bgp vrf CUSTOMER-A 150.23.6.6
```
![](images/6.2_1_shBgp.png)
>NOTE:
>Output from R2 is similar, omitted for brevity.

<br/><br/>

## Task 6.3: Configure color to avoid using links with marked affinity (red link)

To use the affinity constraints, we will need to color the links that we want to reference to exclude or include in our path, we achieve this by adding a name to the links and then providing an ID to that name by using bit-position.

On R1 and R2 issue the following command:
```
segment-routing
 traffic-eng
  interface GigabitEthernet0/0/0/3
   affinity
    name RED
   !
  !
  affinity-map
   name RED bit-position 1
  !
 !
!
```
<br/><br/>

## Task 6.4: Configure SRTE policy to avoid red links

On R1 and R2 issue the following command:
```
segment-routing
 traffic-eng
  policy AFF-TE_COLOR-3236_R7
   color 3236 end-point ipv4 7.7.7.7
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type te
      !
     !
     constraints
      affinity
       exclude-any
        name RED
       !
      !
     !
 
root
segment-routing
 traffic-eng
  policy AFF-TE_COLOR-3236_R8
   color 3236 end-point ipv4 8.8.8.8
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type te
      !
     !
     constraints
      affinity
       exclude-any
        name RED
       !
      !
     !
    !
   !
  !
 !
!
```
<br/><br/>

## Task 6.5: Verify service Path

On R1 & R2 display the BGP prefix.
```
sh bgp vrf CUSTOMER-A 150.23.6.6
```
![](images/6.5_1_shBgp.png)


On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 3236
```
![](images/6.5_2_shSegTE.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 

On R1 & R2, display cef for CE3 Loopback 36 (150.23.6.6).
```
sh cef vrf CUSTOMER-A 150.23.6.6
```
![](images/6.5_3_shCef.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display the binding SID info.
```
RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119038**
```
![](images/6.5_4_shMplsLab.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On CE1, traceroute to CE3 to display the new path taken.
```
traceroute vrf CUSTOMER-A 150.23.6.6 probe 1
```
![](images/6.5_5_tracert.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. 
<br/><br/>

# Scenario 7 – Intra and Inter-Domain Network Slicing for On-Demand Next Hop (ODN)

Segment Routing On-Demand Next Hop (ODN) allows a service head-end router to automatically instantiate an SR policy to a BGP next-hop when required (on-demand).

For this scenario, we will be using CE2 Loopback 37 (150.22.7.7) and CE3 Loopback37 (150.23.7.7) both will use the color 3207

![](images/7.0_diagram.png)
<br/><br/>

## Task 7.1: Configure color for ODN

Create the extended community and update the route-policy.

Apply the following configuration in R4:
```
extcommunity-set opaque COLOR-3207
  3207
end-set
!
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3222 #####
  if destination in (150.22.2.2) then
    set extcommunity color COLOR-3222
    ##### ODN - Latency #####
  elseif destination in (150.22.7.7) then
    set extcommunity color COLOR-3207
    ##### Everything Else #####
  else
    pass
  endif
end-policy
```

Apply the following configuration in R7 and R8:
```
extcommunity-set opaque COLOR-3207
  3207
end-set
!
route-policy CUST-A_SET_COLOR_IN
  ##### Explicit Path - Color 3232 #####
  if destination in (150.23.2.2) then
    set extcommunity color COLOR-3232
    ##### Dynamic Path - Latency #####
  elseif destination in (150.23.3.3) then
    set extcommunity color COLOR-3233
    ##### Dynamic Path - TE #####
  elseif destination in (150.23.4.4) then
    set extcommunity color COLOR-3234
    ##### Anycast SID - TE #####
  elseif destination in (150.23.5.5) then
    set extcommunity color COLOR-3235
    ##### Affinity - TE #####
  elseif destination in (150.23.6.6) then
    set extcommunity color COLOR-3236
    ##### ODN - Latency #####
  elseif destination in (150.23.7.7) then
    set extcommunity color COLOR-3207
    ##### Everything Else #####
  else
    pass
  endif
end-policy
```

>NOTE: There is no need to apply the route-policy in R4, R7 & R8 towards CE3 since that was done in scenario 2 already.
<br/><br/>

## Task 7.2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following commands:
```
sh bgp vrf CUSTOMER-A 150.22.7.7
sh bgp vrf CUSTOMER-A 150.23.7.7
```
![](images/7.2_1_shBgp.png)

>NOTE:
>Output from R2 is similar, omitted for brevity.
<br/><br/>

## Task 7.3: Configure SRTE policy to use ODN vs defined end point.

The configuration of SR-TE ODN does not use the end-point IP, instead it dynamically creates the path based on the received BGP routes next hop, then the head-end will contact the PCE and request the path towards the destination that minimize the cumulative latency

On R1 and R2 issue the following command:
```
segment-routing
 traffic-eng
  on-demand color 3207
   dynamic
    pcep
    !
    metric
     type latency
    !
   !
  !
 !
!
```
<br/><br/>

## Task 7.4: Verify Service Path

On R1 & R2 display the BGP prefixes.
```
sh bgp vrf CUSTOMER-A 150.22.7.7
```
![](images/7.4_1_shBgp.png)


```
sh bgp vrf CUSTOMER-A 150.23.7.7
```
![](images/7.4_2_shBgp.png)


On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 3207
```
![](images/7.4_3_shSegTE.png)
![](images/7.4_4_shSegTE.png)



>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 

On R1 & R2, display cef for 150.22.7.7 and 150.23.7.7
```
sh cef vrf CUSTOMER-A 150.22.7.7
sh cef vrf CUSTOMER-A 150.23.7.7
```
![](images/7.4_5_shCef.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display the binding SID info
```
sh mpls forwarding labels 119042
sh mpls forwarding labels 119043
```
![](images/7.4_6_shMpls.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 

On CE1, traceroute to CE2 and CE3 to display the new path taken.
```
traceroute vrf CUSTOMER-A 150.22.7.7 probe 1
traceroute vrf CUSTOMER-A 150.23.7.7 probe 1
```
![](images/7.4_7_tracert.png)


>NOTE:
>Your lab output may be different if the ECMP hashed R1 instead, output using R1 is omitted for brevity. 
<br/><br/>

# Scenario 8 - Configure Flex Algo & ODN for low latency

SR IGP Flex Algo:

- Complements the SRTE solution by allowing operators to define one or several flexible algorithms with optimization objectives and a set of constraints
  - Minimize igp-metric or delay or te-metric
- Leverages the SRTE benefits of simplicity and automation
  - Automated sub-50nsec FRR (TILFA)
  - On-Demand Policy (ODN)
  - Automated Steering (AS)
- An IGP Prefix-Sid is associated to a prefix and an algorithm. The default algorithm (0) is the IGP shortest path. Algorithms 0 to 127 are reserved for standardized (IETF/IANA) algorithms.
- Algorithms 128 to 255 are customized by the operator. They are called SR IGP Flexible Algorithms ("Flex-Algo")
- Operational Simplicity and Scale
  - Multiple prefix-SIDs of different algorithms can share the same loopback address
  - Only nodes participating in a Flex-Algo compute the paths to the prefix-SIDs of that Flex-Algo
  - Prefix-SIDs of any algorithm allows SRTE paths to be expressed in a better way by reducing the number of SIDs required in the SID list
- Frequent use-cases include low-delay routing and dual-plane disjoint paths

Below section covers the following tasks:

1. Configure Flex Algo 128 (low latency) and a prefix SID on the ABRs (R3, R4, R5 & R6)
2. Configure Flex Algo 128 (low latency) and a Prefix SID on all the remaining ISIS Nodes
3. Verify Flex Algo 128 on all ISIS nodes
4. Configure RPL to color CE prefixes of CUSTOMER-B on R4, R7 & R8
5. Verify the prefix is tagged with the new color
6. Configure ODN and flex Algo 128 policy on R1 & R2
7. Verify Service Path
8. Change Latency value of a link
9. Verify Service Path Change
10. Restore original Latency value

![](images/8.0_diagram.png)

Below table summarize Loopback0 Prefix-SID that will be configured in the following section.

| **Node** | **Flex Algo 128 - Prefix-SID** |
| --- | --- |
| R1 | 101128 |
| R2 | 102128 |
| R3 | 103128 |
| R4 | 104128 |
| R5 | 105128 |
| R6 | 106128 |
| R7 | 107128 |
| R8 | 108128 |
| PCE1 | 111128 |
| PCE2 | 112128 |
| PCE3 | 113128 |

<br/><br/>

## Task 8.1: Configure Flex Algo 128 for low latency and a prefix SID on the ABR Nodes

Configure the below commands on R3, R4, R5 & R6:

Apply the following configuration in R3:
```
router isis ACCESS-1
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 103128
  !
 !
!
router isis AGG-CORE
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 103128
  !
 !
```

Apply the following configuration in R4:

```
router isis ACCESS-1
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 104128
  !
 !
!
router isis AGG-CORE
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 104128
  !
 !
```
Apply the following configuration in R5:
```
router isis ACCESS-2
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 105128
  !
 !
!
router isis AGG-CORE
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 105128
  !
 !
```

Apply the following configuration in R6:
```
router isis ACCESS-2
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 106128
  !
 !
!
router isis AGG-CORE
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 106128
  !
 !
```
<br/><br/>

## Task 8.2: Configure Flex Algo 128 for low latency and a prefix SID on all the remaining ISIS nodes

Configure the below commands on R1, R2, PCE1, PCE2, PCE3, R7 & R8

Apply the following configuration in R1:
```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 101128
  !
 !
```

Apply the following configuration in R2:
```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 102128
  !
 !
```

Apply the following configuration in PCE1:
```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 111128
  !
 !
```

Apply the following configuration in PCE2:
```
router isis AGG-CORE
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 112128
  !
 !
```

Apply the following configuration in PCE3:
```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 113128
  !
 !
```


Apply the following configuration in R7:
```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 107128
  !
 !
```


Apply the following configuration in R8:
```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 108128
  !
 !
```
<br/><br/>

## Task 8.3: Verify Flex Algo 128 on all ISIS nodes

Issue the following command on R1, R2 & PCE1:
```
sh isis flex-algo 128
```

![](images/8.3_1.png)

>NOTE:
>Output from R2 & PCE1 are similar, omitted for brevity. 

Issue the following command on R3, R4 & PCE2:
```
sh isis flex-algo 128
```

![](images/8.3_2.png)

>NOTE:
>Output from R4 & PCE2 are similar, omitted for brevity.

Issue the following commands on R5, R6, PCE3, R7 & R8:
```
show isis flex-algo 128
```

![](images/8.3_3.png)

>NOTE:
>Output from R6,PCE3 & R8 are similar, omitted for brevity. 
<br/><br/>

## Task 8.4: RPL to Color CE prefixes of VRF CUSTOMER-B on R4, R7 & R8

Apply the following configuration in R4:
```
extcommunity-set opaque COLOR-4128
  4128
end-set
!
route-policy CUST-B_SET_COLOR_IN
  ##### ODN Flex Algo 128 - Latency #####
  if destination in (150.22.7.7) then
    set extcommunity color COLOR-4128
    ##### Everything Else #####
  else
    pass
  endif
end-policy
!
router bgp 65001
 vrf CUSTOMER-B
  neighbor 172.4.22.101
   address-family ipv4 unicast
    route-policy CUST-B_SET_COLOR_IN in
   !
  !
 !
!
```


Apply the following configuration in R7:
```
extcommunity-set opaque COLOR-4128
  4128
end-set
!
route-policy CUST-B_SET_COLOR_IN
  ##### ODN Flex Algo 128 - Latency #####
  if destination in (150.23.7.7) then
    set extcommunity color COLOR-4128
    ##### Everything Else #####
  else
    pass
  endif
end-policy
!
router bgp 65001
 vrf CUSTOMER-B
  neighbor 172.7.23.101
   address-family ipv4 unicast
    route-policy CUST-B_SET_COLOR_IN in
   !
  !
 !
!
```

Apply the following configuration in R8:
```
extcommunity-set opaque COLOR-4128
  4128
end-set
!
route-policy CUST-B_SET_COLOR_IN
  ##### ODN Flex Algo 128 - Latency #####
  if destination in (150.23.7.7) then
    set extcommunity color COLOR-4128
    ##### Everything Else #####
  else
    pass
  endif
end-policy
!
router bgp 65001
 vrf CUSTOMER-B
  neighbor 172.8.23.101
   address-family ipv4 unicast
    route-policy CUST-B_SET_COLOR_IN in
   !
  !
 !
!
```
<br/><br/>

## Task 8.5: Verify the prefix is tagged with the new color

On our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color (4128).

On R1 & R2 issue the following commands:
```
sh bgp vrf CUSTOMER-B 150.22.7.7
sh bgp vrf CUSTOMER-B 150.23.7.7
```
![](images/8.5_1.png)

>NOTE: Output from R2 is similar, omitted for brevity.
<br/><br/>

## Task 8.6: ODN and flex Algo 128 policy on R1 & R2

Configure SR Policy with ODN and Flex Algo on the head end R1 and R2.

Apply the following configuration in R1 & R2:
```
segment-routing
 traffic-eng
  on-demand color 4128
   dynamic
    pcep
    !
    sid-algorithm 128
   !
  !
 !
!
```
<br/><br/>

## Task 8.7: Verify Service Path

On R1 & R2 display the BGP prefixes.
```
sh bgp vrf CUSTOMER-B 150.22.7.7
```
![](images/8.7_1.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output
```
sh segment-routing traffic-eng policy color 4128
```
![](images/8.7_2.png)
![](images/8.7_3.png)
>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab.

On R1 & R2, display cef for 150.22.7.7 and 150.23.7.7
```
sh cef vrf CUSTOMER-B 150.22.7.7
```
![](images/8.7_4.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 

On R1 & R2, display the binding SID info
```
sh mpls forwarding labels 119072
sh mpls forwarding labels 119075
```
![](images/8.7_5.png)

>NOTE:
>Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. 


On CE1, traceroute to CE3 to display the path taken.

Below traceroute follow **inter domain path** using labels from ODN and Flex algo 128 policy on R1 and R2 covered in previous steps

```
traceroute vrf CUSTOMER-B 150.23.7.7 probe 1
```

![](images/8.7_6.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. 

##

Below traceroute follow **intra domain path using** labels fromODN and Flex algo 128 policy on R1 and R2 covered in previous steps.
```
traceroute vrf CUSTOMER-B 150.22.7.7 probe 1
```

![](images/8.7_7.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. 

Prefix-SIDs of Flex algorithm allows SRTE paths to be expressed in a better way by reducing the number of SID's required in the SID list.

Below Section compares the SID LIST of ODN SR Policy (Color 3207) and Flex Algo + ODN SR policy (Color 4128).
```
show segment-routing traffic-eng policy color 3207 endpoint ipv4 7.7.7.7 | in  Prefix-SID 
show segment-routing traffic-eng policy color 4128 endpoint ipv4 7.7.7.7 | in  Prefix-SID 
show segment-routing traffic-eng policy color 3207 endpoint ipv4 8.8.8.8 | in  Prefix-SID 
show segment-routing traffic-eng policy color 4128 endpoint ipv4 8.8.8.8 | in  Prefix-SID 
```
![](images/8.7_8.png)
<br/><br/>

## Task 8.8: Change Latency value of a link

In this task, we will temporarily make changes to the delay of PCE2-R5 and PCE2-R6 to simulate a degradation of those links which will trigger a re-optimization of the path.

On PCE2 configure the delay to R5 and R6 as 150ms.
```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 150
  !
 !
 interface GigabitEthernet0/0/0/3
  delay-measurement
   advertise-delay 150
  !
 !
!
```
<br/><br/>


## Task 8.9: Verify Service Path Change

The new SR-TE path should re-optimize to avoid the high delay links.

![](images/8.9_Diagram.png)

On R1 & R2 display the SR-TE policy details.
```
sh segment-routing traffic-eng policy color 4128
```
![](images/8.9_1.png)
>NOTE:
>Output from R2 is similar & Output for SR Policy to 4.4.4.4 is unaffected, omitted for brevity. 

On CE1, traceroute to CE3 to display the new path taken.
```
traceroute vrf CUSTOMER-B 150.23.7.7 probe 1
```
![](images/8.9_2.png)

>NOTE:
>Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. 
<br/><br/>

## Task 8.10: Restore original Latency value

On PCE2 restore the delay to R5 as 10ms and to R6 as 13ms.
```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 10
  !
 !
 interface GigabitEthernet0/0/0/3
  delay-measurement
   advertise-delay 13
  !
 !
!
```
<br/><br/>

# Scenario 10 - Configure Flex Algo for Dual Plane

One use case of Flex-Algo involve dual plane disjoint paths. Below topology consists of two planes:

User Plane 1 – Flex Algo 129 – PCE1, R3, PCE2, R5, PCE3

User Plane 2 – Flex Algo 130 – PCE1, R4, PCE2, R6, PCE3

Both algorithms have the same definition: optimize IGP metric, no constraints. By using Flex Algorithm, operators can also join completely disjoint path. However, in the below topology devices PCE1, PCE2, PCE3 & Services nodes R1, R2, R7, R8 are participating in both user planes 1 & 2.

![](images/446d077a454a7cca.png)

| **Node** | **Flex Algo 129 - Prefix-SID** | **Flex Algo 130 - Prefix-SID** |
| --- | --- | --- |
| R1 | 101129 | 101130 |
| R2 | 102129 | 102130 |
| R3 | 103129 | ------- |
| R4 | ------ | 104130 |
| R5 | 105129 | ------ |
| R6 | ------ | 106130 |
| R7 | 107129 | 107130 |
| R8 | 108129 | 108130 |
| PCE1 | 111129 | 111130 |
| PCE2 | 112129 | 112130 |
| PCE3 | 113129 | 113130 |

## Task 1: Configure Flex Algo for User plane-1

Follow below steps to configure User-plane -1 on devices R3 & R5.

Apply the following configuration in R3:

**router isis ACCESS-1**

**flex-algo 129**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 103129**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 129**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 103129**

**!**

**!**

**!**

Apply the following configuration in R5:

**router isis ACCESS-2**

**flex-algo 129**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 105129**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 129**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 105129**

**!**

**!**

**!**

## Task 2: Configure Flex Algo for User plane-2

Follow below steps to configure User-plane -2 on devices R4 & R6

Apply the following configuration in R4:

**router isis ACCESS-1**

**flex-algo 130**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 130 absolute 104130**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 130**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 130 absolute 104130**

**!**

**!**

**!**

Apply the following configuration in R6:

**router isis ACCESS-2**

**flex-algo 130**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 130 absolute 106130**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 130**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 130 absolute 106130**

**!**

**!**

**!**

## Task 3: Configure Flex Algo for User plane-1 & 2

Follow below steps to configure both user planes on devices R1, R2, PCE1, PCE2, PCE3, R7 and R8.

Apply the following configuration in R1:

**router isis ACCESS-1**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 101129**

**prefix-sid algorithm 130 absolute 101130**

**!**

**!**

Apply the following configuration in R2:

**router isis ACCESS-1**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 102129**

**prefix-sid algorithm 130 absolute 102130**

**!**

**!**

Apply the following configuration in PCE1:

**router isis ACCESS-1**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 21029**

**prefix-sid algorithm 130 absolute 21030**

**!**

**!**

Apply the following configuration in PCE2:

**router isis AGG-CORE**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 112129**

**prefix-sid algorithm 130 absolute 112130**

**!**

**!**

Apply the following configuration in PCE3:

**router isis ACCESS-2**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 113129**

**prefix-sid algorithm 130 absolute 113130**

**!**

**!**

**!**

Apply the following configuration in R7:

**router isis ACCESS-2**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 107129**

**prefix-sid algorithm 130 absolute 107130**

**!**

**!**

##

Apply the following configuration in R8:

**router isis ACCESS-2**

**flex-algo 129**

**flex-algo 130**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 129 absolute 108129**

**prefix-sid algorithm 130 absolute 108130**

**!**

**!**

## Task 4: Verify Flex Algo 129 on User-Plane1 Nodes

Run the command on following devices R1, R3 & R7:

RP/0/RP0/CPU0:R1# **sh isis flex-algo 129**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 129:

Level-1:

Definition Priority: 128

Definition Source: R3.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

RP/0/RP0/CPU0:R3# **show isis flex-algo 129**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 129:

Level-1:

Definition Priority: 128

Definition Source: R3.00, (Local)

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

IS-IS AGG-CORE Flex-Algo Database

Flex-Algo 129:

Level-1:

Definition Priority: 128

Definition Source: R5.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

RP/0/RP0/CPU0:R7# **show isis flex-algo 129**

IS-IS ACCESS-2 Flex-Algo Database

Flex-Algo 129:

Level-1:

Definition Priority: 128

Definition Source: R5.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2, PCE1 & R5 is similar, omitted for brevity. |

## Task 5: Verify Flex Algo 130 on User-Plane 2 Nodes

Run the command on following devices R1, R4 & R7:

RP/0/RP0/CPU0:R1# **sh isis flex-algo 130**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 130:

Level-1:

Definition Priority: 128

Definition Source: R4.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

RP/0/RP0/CPU0:R4# **sh isis flex-algo 130**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 130:

Level-1:

Definition Priority: 128

Definition Source: R4.00, (Local)

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

IS-IS AGG-CORE Flex-Algo Database

Flex-Algo 130:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

RP/0/RP0/CPU0:R7# **sh isis flex-algo 130**

IS-IS ACCESS-2 Flex-Algo Database

Flex-Algo 130:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: Yes

Definition Metric Type: IGP

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from other nodes, omitted for brevity. |

## Task 6: Configure RPL to Color CE prefixes of VRF CUSTOMER-B

Configure RPL on R7 & R8 as per below table.

|
 |
 |
| --- | --- |
| **Flex Algo** | **Color** |
| **129** | **4129** |
| **130** | **4130** |

Apply the following configuration in R7:

**extcommunity-set**  **opaque COLOR-4129**

**4129**

**end-set**

**!**

**extcommunity-set opaque COLOR-4130**

**4130**

**end-set**

**!**

**route-policy CUST-B\_SET\_COLOR\_IN**

**##### ODN Flex Algo 128 - Latency #####**

**if destination in (150.23.7.7) then**

**set extcommunity color COLOR-4128**

**#####**  **ODN Flex Algo 129 - User Plane1 #####**

**elseif destination in** **(150.23.1.1)**  **then**

**set extcommunity color COLOR-4129**

**##### ODN Flex Algo 130 - User Plane2 #####**

**elseif destination in (150.23.2.2) then**

**set extcommunity color COLOR-4130**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

Apply the following configuration in R8:

**extcommunity-set opaque COLOR-4129**

**4129**

**end-set**

**!**

**extcommunity-set opaque COLOR-4130**

**4130**

**end-set**

**!**

**route-policy CUST-B\_SET\_COLOR\_IN**

**##### ODN Flex Algo 128 - Latency #####**

**if destination in (150.23.7.7) then**

**set extcommunity color COLOR-4128**

**##### ODN Flex Algo 129 - User Plane1 #####**

**elseif destination in** **(150.23.1.1)**  **then**

**set extcommunity color COLOR-4129**

**##### ODN Flex Algo 130 - User Plane2 #####**

**elseif destination in (150.23.2.2) then**

**set extcommunity color COLOR-4130**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

## Task 7: Verify the prefix is tagged with the new color

On our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color (4129, 4130).

On R1 & R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.23.1.1**

BGP routing table entry for 150.23.1.1/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 125 125

Last Modified: May 4 03:02:09.059 for 00:01:59

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119012

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 125

Extended community: Color:4129 RT:65001:4

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:4

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119001

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:4129 RT:65001:4

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:4

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from Color 4130 and R2 are similar, omitted for brevity. |

## Task 8: ODN and flex Algo 129 & 130 policy on R1 & R2

Configure SR Policy with ODN and Flex Algo on the head end R1 and R2.

Apply the following configuration in R1 & R2:

**segment-routing**

**traffic-eng**

**on-demand color 4129**

**dynamic**

**pcep**

**!**

**sid-algorithm 129**

**on-demand color 4130**

**dynamic**

**pcep**

**!**

**sid-algorithm 130**

**!**

**!**

**!**

## Task 9: Verify Service Path for User Plane 1 - Flex Algo 129

On R1 & R2 display the BGP prefixes.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.23.1.1**

BGP routing table entry for 150.23.1.1/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 209 209

Last Modified: May 10 15:36:11.425 for 00:16:05

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

7.7.7.7 C:4129 (bsid:119088) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119012

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 205

Extended community: Color:4129 RT:65001:4

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4129, up, registered, bsid 119088, if-handle 0x000000cc

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:4

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:4129 (bsid:119084) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119014

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:4129 RT:65001:4

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4129, up, registered, bsid 119084, if-handle 0x000000d4

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:4

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output.

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 4129**

SR-TE policy database

---------------------

Color: 4129, End-point: 7.7.7.7

Name: srte\_c\_4129\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:03:46 (since May 4 03:11:13.734)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 129

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4129\_ep\_7.7.7.7\_discr\_100

PLSP-ID: 15

Constraints:

Prefix-SID Algorithm: 129

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 300

103129 [Prefix-SID, 3.3.3.3]

105129 [Prefix-SID, 5.5.5.5]

107129 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119088

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 4129, End-point: 8.8.8.8

Name: srte\_c\_4129\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:03:47 (since May 4 03:11:13.225)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 129

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4129\_ep\_8.8.8.8\_discr\_100

PLSP-ID: 16

Constraints:

Prefix-SID Algorithm: 129

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 400

103129 [Prefix-SID, 3.3.3.3]

105129 [Prefix-SID, 5.5.5.5]

108129 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119084

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display cef for CE3 Loopback 41 (150.23.1.1).

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-B 150.23.1.1**

150.23.1.1/32, version 142, internal 0x5000001 0x30 (ptr 0xd7ae320) [1], 0x0 (0xe1dd1d8), 0xa08 (0xec023c8)

Updated May 10 15:36:11.580

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119088, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f0e40 0x0]

recursion-via-label

next hop VRF – 'default', table – 0xe0000000

next hop via 119088/0/21

next hop srte\_c\_4129\_ labels imposed {ImplNull 119012}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info.

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119088**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119088 Pop No ID srte\_c\_4129\_ point2point 160

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

## Task 10: Verify Service Path for User Plane 2 - Flex Algo 130

On R1 & R2 display the BGP prefix.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.23.2.2**

BGP routing table entry for 150.23.2.2/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 210 210

Last Modified: May 10 15:36:11.425 for 00:26:43

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

7.7.7.7 C:4130 (bsid:119089) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119013

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 206

Extended community: Color:4130 RT:65001:4

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4130, up, registered, bsid 119089, if-handle 0x000000dc

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:4

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:4130 (bsid:119090) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119015

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:4130 RT:65001:4

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4130, up, registered, bsid 119090, if-handle 0x000000e4

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:4

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output.

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 4130**

SR-TE policy database

---------------------

Color: 4130, End-point: 7.7.7.7

Name: srte\_c\_4130\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:08:37 (since May 4 03:11:13.734)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 130

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4130\_ep\_7.7.7.7\_discr\_100

PLSP-ID: 17

Constraints:

Prefix-SID Algorithm: 130

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 500

104130 [Prefix-SID, 4.4.4.4]

106130 [Prefix-SID, 6.6.6.6]

107130 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119089

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 4130, End-point: 8.8.8.8

Name: srte\_c\_4130\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:08:37 (since May 4 03:11:13.225)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 130

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4130\_ep\_8.8.8.8\_discr\_100

PLSP-ID: 18

Constraints:

Prefix-SID Algorithm: 130

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 400

104130 [Prefix-SID, 4.4.4.4]

106130 [Prefix-SID, 6.6.6.6]

108130 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119090

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display cef for CE3 Loopback 42 (150.23.2.2).

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-B 150.23.2.2**

150.23.2.2/32, version 144, internal 0x5000001 0x30 (ptr 0xd7ae248) [1], 0x0 (0xe1dd190), 0xa08 (0xec024e8)

Updated May 10 15:36:11.580

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119089, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f5100 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119089/0/21

next hop srte\_c\_4130\_ labels imposed {ImplNull 119013}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info.

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119089**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119089 Pop No ID srte\_c\_4130\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 to display the path taken.

Below traceroute follow **User Plane -1** using labels from ODN and Flex algo 129 policy on R1 and R2 covered in previous steps

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-B 150.23.1.1 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.1.1

1 r1 (172.1.21.100) 0 msec

2 r3 (172.1.3.1) [MPLS: Labels 105129/107129/119012 Exp 0] 0 msec

3 r5 (172.3.5.1) [MPLS: Labels 107129/119012 Exp 0] 0 msec

4 r7 (172.5.7.1) [MPLS: Label 119012 Exp 0] 0 msec

5 ce3 (172.7.23.101) 0 msec

|

! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

##

Below traceroute follow **User Plane-2 using** labels fromODN and Flex algo 130 policy on R1 and R2 covered in previous steps.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-B 150.23.2.2 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.2.2

1 r1 (172.1.21.100) 0 msec

2 r2 (172.1.2.1) [MPLS: Labels 104130/106130/107130/119013 Exp 0] 19 msec

3 r4 (172.4.11.0) [MPLS: Labels 106130/107130/119013 Exp 0] 29 msec

4 r6 (172.4.6.1) [MPLS: Labels 107130/119013 Exp 0] 9 msec

5 r8 (172.6.8.1) [MPLS: Labels 107130/119013 Exp 0] 9 msec

6 r7 (172.7.8.0) [MPLS: Label 119013 Exp 0] 0 msec

7 ce3 (172.7.23.101) 9 msec

|

! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

##

6 ![](images/7e7316a3bd6bf21f.png)| Page
