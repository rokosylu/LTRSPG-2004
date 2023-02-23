

![](images/cisco-live.png)ADD CISCO LIVE IMAGE WHEN AVAILABLE

# -2023

---

<font size= "7"> **Programmable Intent-Based Slicing, Powered by Segment Routing on IOS XR** </font>

---

<font size= "7"> **LTRSPG-2004** </font>

---

**Speakers:**

**Rob Kosyluk – Customer Delivery Architect**

**Pradeep Malik – Customer Delivery Architect**

**Sam Yagui – Customer Delivery Architect**

![](images/cisco-live-2.png) ADD CISCO LIVE IMAGE WHEN AVAILABLE

---
# Table of Content

Use GitHub TOC

![](images/) ADD capture of TOC on GITHUB

---

# Learning Objectives

Upon completion of this lab, you will be able to:

- Describe network slicing methodologies to partition the network infrastructure to provide differentiated services for different SLAs.
- Deploy On-Demand Next-Hop (ODN) policies with Automated Steering (AS) to enable auto-steering of traffic based on the operator SLAs.
- Use Flex-Algo with On-Demand Next-Hop and Automated Steering to allow auto-steering of traffic into a topology, or path, defined by a service provider's logic.

<br/>

>NOTE:
>This lab familiarizes the user with the Cisco SR TE and its configuration.Although the lab design and configuration examples could be used as a reference, it's not a real design, so not all recommended features are used, or enabled optimally. For design related questions please contact your Cisco representative, or a Cisco partner.  

<br/>
<br/>

---

# Introduction

Segment routing for traffic engineering (SR-TE) uses a "policy" to steer traffic through the network. An SR-TE policy path is expressed as a list of segments that specifies the path, called a segment ID (SID) list. Each segment is an end-to-end path from the source to the destination, and instructs the routers in the network to follow the specified path instead of following the shortest path calculated by the IGP. If a packet is steered into an SR-TE policy, the SID list is pushed on the packet by the head-end. The rest of the network executes the instructions embedded in the SID list.

An SR-TE policy is identified as an ordered list (head-end, color, end-point):

- Head-end – Where the SR-TE policy is instantiated
- Color – A numerical value that distinguishes between two or more policies to the same node pairs (Head-end – End point)
- End-point – The destination of the SR-TE policy

Every SR-TE policy has a color value. Every policy between the same node pairs requires a unique color value.

SR-TE is the foundation of network slicing.

## Overview

In this lab, you will configure and verify SR-TE slicing in a SR-MPLS network by using the following tools:

- Explicit Path
- Automated Steering with latency, te metric, and affinity constraints.
- ODN (On-Demand NextHop)
- Flex-Algo

## Physical Topology Diagram

Drawing of the physical connections in the lab.

![](images/physicalTopology.png)

## IGP (IS-IS) Topology Diagram

There are 3 different IGP domains "ACCESS 1", "AGG-CORE" and "ACCESS 2" in the lab. There is no redistribution between areas. BGP-LU is used across the domains for end to end loopback reachability.

![](images/IGPTopology.png)

## IP and SID Diagram

The SRGB in the lab is configured as 19000-119000. There are 2 VRFs configured "CUSTOMER-A" and "CUSTOMER-B".

![](images/IPandSID.png)

## IGP Metric Diagram

The default IGP metric is 100 throughout the topology.

![](images/IGPMetric.png)

## Latency Metric Diagram

The default latency metrics of the links are seen below. Latency is lower on the links going through the inline PCE nodes.

![](images/LatencyMetric.png)

##

## TE Metric Diagram

The default TE metric is 1000 throughout the topology.

![](images/TEMetric.png)

# Access the Lab

To start, find the anyconnect icon on the taskbar or home screen.

![](RackMultipart20230222-1-cgwn2a_html_14ce9d0a18fb7108.png)

Double click on the icon to open the Cisco AnyConnect Mobility Client.

![](RackMultipart20230222-1-cgwn2a_html_325ba1b133700f9e.png)

Connect to your POD. The format will be [https://64.100.9.4/CLPODx](https://64.100.9.4/CLPODx) where X is your POD number.

![](RackMultipart20230222-1-cgwn2a_html_cd00f0ae868d297d.png)

Once connected a security warning message will be seen. Click "Connect Anyway".

![](RackMultipart20230222-1-cgwn2a_html_46a2b563343ce9ea.png)

Enter the username and password for your pod.

![](RackMultipart20230222-1-cgwn2a_html_95dab4b1258cd5cd.png)

Click accept to connect to the lab.

![](RackMultipart20230222-1-cgwn2a_html_b79d42209b771423.png)

Once connected you will see this popup window.

![](RackMultipart20230222-1-cgwn2a_html_7049bb8d24cac6c3.png)

Now you are ready to start the lab 

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

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 Where XX is your POD number. i.e: POD01 will use 01, POD 13 will use 13Username and Password is cisco/cisco for all devices |



NOTE

 |
 | For convenience all the lab configuration sections are posted in the following link [https://github.com/rokosylu/LTRSPG-2004](https://github.com/rokosylu/LTRSPG-2004) which will enable to easly copy and paste the config to your lab devices |

# Scenario 1 - Lab Verification of the Underlay and Overlay Fundamentals

All IP addresses, IGP protocol configuration, and basic BGP configuration have been completed in the simulation. The purpose of this is for you to focus on SRTE and other advanced topics. However, let's verify all protocols are functional properly, using this as a quick review of the fundamentals as well.

## Task 1: Verify ISIS Operation

Log into R3 and verify it has an adjacency relationship with R1, R4, and PCE1 in the ACCESS-1 process and R4, R5, and PCE2 in the AGG-CORE Process. There are two processes since this is an ABR.

RP/0/RP0/CPU0:R3# **show isis adjacency**

IS-IS ACCESS-1 Level-1 adjacencies:

System Id Interface SNPA State Hold Changed NSF IPv4 IPv6

BFD BFD

R1 Gi0/0/0/3 \*PtoP\* Up 29 00:01:55 Yes Up None

R4 Gi0/0/0/1.34 \*PtoP\* Up 21 00:33:14 Yes Up None

PCE1 Gi0/0/0/2 \*PtoP\* Up 21 00:32:44 Yes Up None

Total adjacency count: 3

IS-IS AGG-CORE Level-1 adjacencies:

System Id Interface SNPA State Hold Changed NSF IPv4 IPv6

BFD BFD

R4 Gi0/0/0/1.43 \*PtoP\* Up 26 00:33:14 Yes Up None

R5 Gi0/0/0/5 \*PtoP\* Up 24 00:33:07 Yes Up None

PCE2 Gi0/0/0/4 \*PtoP\* Up 25 00:32:42 Yes Up None

You can verify the other ABRs (R4, R5, R6) have adjacencies in both processes.

## Task 2: Verify Segment Routing Configuration and Operation

IOS-XR has a default SRBG of 16000-23999. In this lab, we are not using the default SRBG but rather have configured one from 19000-119000. Verify this is configured correctly on R3.

RP/0/RP0/CPU0:R3# **show run segment-routing**

segment-routing

global-block 19000 119000

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | When changing the SRGB, a reload of the router may be needed. |

Verify the size of the SRGB on R3 is the same as what is configured. It starts at 19000 and has a size of 100001 labels.

RP/0/RP0/CPU0:R3# **sh mpls label table label 19000 detail**

Table Label Owner State Rewrite

----- ------- ------------------------------- ------ -------

0 19000 ISIS(A):AGG-CORE InUse No

ISIS(A):ACCESS-1 InUse No

(Lbl-blk SRGB, vers:0, (start\_label=19000, size=100001)

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

RP/0/RP0/CPU0:R3# **show run router isis**

router isis ACCESS-1

set-overload-bit on-startup 360

is-type level-1

net 49.0220.0000.0000.0003.00

distribute link-state instance-id 100

nsf ietf

log adjacency changes

lsp-gen-interval maximum-wait 5000 initial-wait 50 secondary-wait 200

lsp-mtu 4352

address-family ipv4 unicast

metric-style wide

metric 100

microloop avoidance segment-routing

mpls traffic-eng router-id Loopback0

spf-interval maximum-wait 5000 initial-wait 50 secondary-wait 200

segment-routing mpls sr-prefer

!

!

interface Loopback0

address-family ipv4 unicast

tag 100

prefix-sid index 3

Check the ISIS segment-routing label table and verify 19003 is the label allocated for the local loopback0 interface. Recall R3 is an ABR and the interface/label will be in both processes.

RP/0/RP0/CPU0:R3# **show isis segment-routing label table**

IS-IS ACCESS-1 IS Label Table

Label Prefix/Interface

---------- ----------------

19001 1.1.1.1/32

19002 2.2.2.2/32

19003 Loopback0

19004 4.4.4.4/32

19011 11.11.11.11/32

IS-IS AGG-CORE IS Label Table

Label Prefix/Interface

---------- ----------------

19003 Loopback0

19004 4.4.4.4/32

19005 5.5.5.5/32

19006 6.6.6.6/32

19012 12.12.12.12/32

## Task 3: Verify Link Protection (TI-LFA)

Topology-Independent Loop-Free Alternate (TI-LFA) uses segment routing to provide link, node, and Shared Risk Link Groups (SRLG) protection in topologies where other fast reroute techniques cannot provide protection.

TI-LFA is enabled under the interfaces in ISIS. Run the following command to verify the configuration.

RP/0/RP0/CPU0:R3# **show run router isis ACCESS-1 interface**

router isis ACCESS-1

!

interface Loopback0

address-family ipv4 unicast

tag 100

prefix-sid index 3

!

interface GigabitEthernet0/0/0/1.34

bfd minimum-interval 300

bfd multiplier 4

bfd fast-detect ipv4

point-to-point

hello-password hmac-md5 encrypted 104D000A0618

address-family ipv4 unicast

fast-reroute per-prefix

fast-reroute per-prefix ti-lfa

!

interface GigabitEthernet0/0/0/2

bfd minimum-interval 300

bfd multiplier 4

bfd fast-detect ipv4

point-to-point

hello-password hmac-md5 encrypted 110A1016141D

address-family ipv4 unicast

fast-reroute per-prefix

fast-reroute per-prefix ti-lfa

!

interface GigabitEthernet0/0/0/3

bfd minimum-interval 300

bfd multiplier 4

bfd fast-detect ipv4

point-to-point

hello-password hmac-md5 encrypted 14141B180F0B

address-family ipv4 unicast

fast-reroute per-prefix

fast-reroute per-prefix ti-lfa

Checking the details of the primary and backup path the following commands are run.

NOTE: The primary path information is in GREEN and the backup path information is in BLUE.

RP/0/RP0/CPU0:R3# **show isis ipv4 fast-reroute 1.1.1.1/32 detail**

L1 1.1.1.1/32 [200/115] Label: 19001, medium priority

via 172.1.3.0, GigabitEthernet0/0/0/3, Label: ImpNull, R1 tag 100, SRGB Base: 19000, Weight: 0

Backup path: LFA, via 172.3.11.1, GigabitEthernet0/0/0/2, Label: 19001, PCE1, SRGB Base: 19000, Weight: 0, Metric: 300

P: No, TM: 300, LC: No, NP: No, D: No, SRLG: Yes

src R1.00-00, 1.1.1.1, tag 100, prefix-SID index 1, R:0 N:1 P:0 E:0 V:0

L:0, Alg:0

RP/0/RP0/CPU0:R3# **show cef 1.1.1.1/32**

1.1.1.1/32, version 110, labeled SR, internal 0x1000001 0x8310 (ptr 0xdbe72c8) [1], 0x0 (0xe4aa728), 0xa28 (0xf3932d8)

Updated Apr 12 19:11:49.905

remote adjacency to GigabitEthernet0/0/0/3

Prefix Len 32, traffic index 0, precedence n/a, priority 1

via 172.1.3.0/32, GigabitEthernet0/0/0/3, 4 dependencies, weight 0, class 0, protected [flags 0x400]

path-idx 0 bkup-idx 1 NHID 0x0 [0xf6e5b90 0xf6e5aa8]

next hop 172.1.3.0/32

local label 19001 labels imposed {ImplNull}

via 172.3.11.1/32, GigabitEthernet0/0/0/2, 10 dependencies, weight 0, class 0, backup (Local-LFA) [flags 0x300]

path-idx 1 NHID 0x0 [0xf2f3e58 0x0]

next hop 172.3.11.1/32

remote adjacency

local label 19001 labels imposed {19001}

The output above shows the regular path sending the traffic to the link between R3-R1 via 171.1.3.0 with a ImpNull label (PHP). The backup path sends to traffic to PCE-1 via 172.3.11.1, and imposes the label of 19001.

## Task 4: Verify BGL-LS

BGP Link-State (LS) is an address family designed to carry an IGP link-state information through BGP. BGP-LS delivers the network topology information to topology servers, or any other application needing to know the topology for decisions.

In this scenario, BGP-LS is used to deliver IGP information between domains. Below is a topology of the BGP-LS peerings.

![](RackMultipart20230222-1-cgwn2a_html_1a672ea0d6fa8214.png)

Verify PCE-1 has a BGP-LS peering to PCE-2 and is receiving all the routes.

RP/0/RP0/CPU0:PCE1# **show bgp link-state link-state summary**

BGP router identifier 11.11.11.11, local AS number 65001

BGP generic scan interval 60 secs

Non-stop routing is enabled

BGP table state: Active

Table ID: 0x0 RD version: 175

BGP main routing table version 175

BGP NSR Initial initsync version 1 (Reached)

BGP NSR/ISSU Sync-Group versions 0/0

BGP scan interval 60 secs

BGP is operating in STANDALONE mode.

Process RcvTblVer bRIB/RIB LabelVer ImportVer SendTblVer StandbyVer

Speaker 175 175 175 175 175 0

Neighbor Spk AS MsgRcvd MsgSent TblVer InQ OutQ Up/Down St/PfxRcd

12.12.12.12 0 65001 271 150 175 0 0 02:27:38 126

Recall BGP-LS carries the IGP information into BGP. Check the IGP information for the link between R7 and R8 on PCE-1.

In the example below, look in the link-state database and search for 172.7.8.1 (the link between R7 and R8). The result will give you two entries. One from R7 and one from R8. Take the whole NRLI and enter into the "show bgp link-state link-state" command to view the details.

RP/0/RP0/CPU0:PCE1# **show bgp link-state link-state | i 172.7.8.1**

Mon Apr 11 21:13:22.501 UTC

\*\>i[E][L1][I0x67][N[c65001][b0.0.0.0][s0000.0000.0007.00]][R[c65001][b0.0.0.0][s0000.0000.0008.00]][L[i172.7.8.0][n172.7.8.1]]/696

\*\>i[E][L1][I0x67][N[c65001][b0.0.0.0][s0000.0000.0008.00]][R[c65001][b0.0.0.0][s0000.0000.0007.00]][L[i172.7.8.1][n172.7.8.0]]/696

RP/0/RP0/CPU0:PCE1#**show bgp link-state link-state [E][L1][I0x67][N[c65001][b0.0.0.0][s0000.0000.0007.00]][R[c65001][b0.0.0.0][s0000.0000.0008.00]][L[i172.7.8.0][n172.7.8.1]]/696**

Mon Apr 11 21:14:36.108 UTC

BGP routing table entry for [E][L1][I0x67][N[c65001][b0.0.0.0][s0000.0000.0007.00]][R[c65001][b0.0.0.0][s0000.0000.0008.00]][L[i172.7.8.0][n172.7.8.1]]/696

Versions:

Process bRIB/RIB SendTblVer

Speaker 156 156

Last Modified: Apr 11 18:42:16.854 for 02:32:19

Paths: (1 available, best #1)

Not advertised to any peer

Path #1: Received by speaker 0

Not advertised to any peer

Local, (Received from a RR-client)

5.5.5.5 (metric 400) from 12.12.12.12 (5.5.5.5)

Origin IGP, localpref 100, valid, internal, best, group-best

Received Path ID 1, Local Path ID 1, version 156

Originator: 5.5.5.5, Cluster list: 12.12.12.12

Link-state: Link ID: Local:8 Remote:8, Local TE Router-ID:

7.7.7.7 Remote TE Router-ID: 8.8.8.8, admin-group: 0x00000000

max-link-bw (kbits/sec): 1000000, max-reserv-link-bw (kbits/sec): 0

max-unreserv-link-bw (kbits/sec): 0 0 0 0 0 0 0 0,

TE-default-metric: 100 metric: 100, ADJ-SID: 119001(70)

ADJ-SID: 119002(30) , ext-admin-group: 0x00000000.0x00000000

.0x00000000.0x00000000.0x00000000.0x00000000.0x00000000

.0x00000000

## Task 5: Verify PCE Communication

All of the SRTE headend work will be done on R1 and R2 in this lab.

RP/0/RP0/CPU0:R1# **show segment-routing traffic-eng pcc ipv4 peer**

Tue Apr 12 20:58:34.791 UTC

PCC's peer database:

--------------------

Peer address: 11.11.11.11, Precedence: 255, (best PCE)

State up

Capabilities: Stateful, Update, Segment-Routing, Instantiation

# Scenario 2 - Inter-Domain Network Slicing for Regular Traffic

In this scenario we will get a baseline of the default routing scheme (IGP shortest path) for vrf "CUSTOMER-A" that we can use it to compare with the other scenarios later

## Task 1: Verify Service Path

On CE1, issue a traceroute in CUSTOMER-A vrf to loopback 31 interface of CE2 and CE3

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.22.1.1 probe 1**

Type escape sequence to abort.

Tracing the route to 150.22.1.1

1 r1 (172.1.21.0) 9 msec

2 r2 (172.1.2.1) [MPLS: Labels 19004/119001 Exp 0] 0 msec

3 r4 (172.4.11.0) [MPLS: Label 119001 Exp 0] 0 msec

4 ce2 (172.4.22.1) 0 msec

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.1.1 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.1.1

1 r1 (172.1.21.0) 9 msec

2 r3 (172.1.3.1) [MPLS: Labels 19007/119008 Exp 0] 0 msec

3 r5 (172.3.5.1) [MPLS: Labels 19007/119008 Exp 0] 0 msec

4 r7 (172.5.7.1) [MPLS: Label 119008 Exp 0] 0 msec

5 ce3 (172.7.23.1) 0 msec


NOTE:
Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

# Scenario 3 - Inter-domain SRTE using Explicit-path

In this scenario we will slice our network using explicit path. We will create two explicit paths, depicted below, the green path (color 3222) from CE1 to CE2 Loopback32 (150.22.2.2) and a red path (color 3232) from CE1 to CE3 Loopback32 (150.23.2.2).

![](RackMultipart20230222-1-cgwn2a_html_17e50e0097672cd5.gif)

## Task 1: Configure color for Explicit-path

We will assign the color for each explicit path at the tail end of each tunnel. The first step is to create an extended community for the colors.

Apply the following configuration in R4:

**extcommunity-set opaque COLOR-3222**

**3222**

**end-set**

Since CE3 has two connections to our network using R7 and R8, we will need to assign color 3232 on both routers.

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3232**

**3232**

**end-set**

After we have created the extended communities, we will need to create the route maps to apply the color to our prefixes, we will apply our route policies in the ingress direction from the CEs.

The route-policy will match our destination IP on the CEs and tag the prefixes with the extended community, then we can use the extended community later to choose the desired path.

Apply the following configuration in R4:

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3222 #####**

**if destination in (150.22.2.2) then**

**set extcommunity color COLOR-3222**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

**router bgp 65001**

**vrf CUSTOMER-A**

**neighbor 172.4.22.1**

**address-family ipv4 unicast**

**route-policy CUST-A\_SET\_COLOR\_IN in**

**!**

**!**

**!**

**!**

Apply the following configuration in R7 and R8:

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path – Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

Apply the following configuration in R7:

**router bgp 65001**

**vrf CUSTOMER-A**

**neighbor 172.7.23.1**

**address-family ipv4 unicast**

**route-policy CUST-A\_SET\_COLOR\_IN in**

**!**

**!**

**!**

**!**

Apply the following configuration in R8:

**router bgp 65001**

**vrf CUSTOMER-A**

**neighbor 172.8.23.1**

**address-family ipv4 unicast**

**route-policy CUST-A\_SET\_COLOR\_IN in**

**!**

**!**

## Task 2: Verify the prefixes are tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefixes to make sure they have been tagged with the new color.

In R1 issue the following commands:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.22.2.2**

BGP routing table entry for 150.22.2.2/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 34 34

Last Modified: Apr 14 00:02:46.705 for 00:02:19

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

4.4.4.4 (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119002

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 34

Extended community: Color:3222 RT:65001:3

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:3

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.2.2**

BGP routing table entry for 150.23.2.2/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 36 36

Last Modified: Apr 14 00:04:35.705 for 00:01:19

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119009

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 35

Extended community: Color:3232 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119011

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3232 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3



NOTE:
Output from R2 is similar, omitted for brevity. |

## Task 3: Configure SRTE policy using Explicit-path

To configure an SR-TE policy with an explicit path, we will need to create the segment list and the SR-TE policy.

The segment-list describes each segment on the path that the packets will need to take. It can use and IP address, an mpls label or a combination of both

An Explicit-path policy is identified by the head-end, color and tail-end tuple, in this task we will be configuring at the head-end router the policy with the assigned color and tail-end (end-point).

On R1 & R2 apply the following configuration:

**segment-routing**

**traffic-eng**

**segment-list PCE1-R3-R4**

**index 10 mpls adjacency 11.11.11.11**

**index 20 mpls adjacency 3.3.3.3**

**index 30 mpls adjacency 4.4.4.4**

**!**

**segment-list R3-R4-R6-R5-R7**

**index 10 mpls label 19003**

**index 20 mpls label 19004**

**index 30 mpls label 19006**

**index 40 mpls label 19005**

**index 50 mpls label 19007**

**!**

**segment-list R4-R3-R5-R6-R8**

**index 10 mpls label 19004**

**index 20 mpls label 19003**

**index 30 mpls label 19005**

**index 40 mpls label 19006**

**index 50 mpls label 19008**

**!**

**policy EXP\_COLOR-3222\_R4**

**color 3222 end-point ipv4 4.4.4.4**

**candidate-paths**

**preference 100**

**explicit segment-list PCE1-R3-R4**

**!**

**!**

**!**

**!**

**policy EXP\_COLOR-3232\_R7**

**color 3232 end-point ipv4 7.7.7.7**

**candidate-paths**

**preference 100**

**explicit segment-list R3-R4-R6-R5-R7**

**!**

**!**

**!**

**!**

**policy EXP\_COLOR-3232\_R8**

**color 3232 end-point ipv4 8.8.8.8**

**candidate-paths**

**preference 100**

**explicit segment-list R4-R3-R5-R6-R8**

**!**

**!**

**!**

**!**

**!**

**!**

## Task 4: Verify Service Path

On R1 & R2 display the BGP prefixes.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.22.2.2**

BGP routing table entry for 150.22.2.2/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 79 79

Last Modified: May 7 04:43:20.425 for 00:20:31

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

4.4.4.4 C:3222 (bsid:119008) (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119013

![Shape1](RackMultipart20230222-1-cgwn2a_html_32a410b690a134b0.gif)

Now SR policy is attached in BGP. This is automated steering,

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 76

Extended community: Color:3222 RT:65001:3

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

SR policy color 3222, up, not-registered, bsid 119008

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:3

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.2.2**

BGP routing table entry for 150.23.2.2/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 80 80

Last Modified: May 7 04:43:20.425 for 00:20:35

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3232 (bsid:119011) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119004

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 77

Extended community: Color:3232 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3232, up, not-registered, bsid 119011

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3232 (bsid:119020) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119002

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3232 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3232, up, not-registered, bsid 119020

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3


### NOTE:
Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3222**

SR-TE policy database

---------------------

Color: 3222, End-point: 4.4.4.4

Name: srte\_c\_3222\_ep\_4.4.4.4

Status:

Admin: up Operational: up for 00:11:36 (since Apr 14 00:12:52.665)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: EXP\_COLOR-3222\_R4

Requested BSID: dynamic

Explicit: segment-list PCE1-R3-R4 (valid)

Weight: 1, Metric Type: TE

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19004 [Prefix-SID, 4.4.4.4]

Attributes:

Binding SID: 119008

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3232**

SR-TE policy database

---------------------

Color: 3232, End-point: 7.7.7.7

Name: srte\_c\_3232\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:12:33 (since Apr 14 00:12:52.665)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: EXP\_COLOR-3232\_R7

Requested BSID: dynamic

Explicit: segment-list R3-R4-R6-R5-R7 (valid)

Weight: 1, Metric Type: TE

19003 [Prefix-SID, 3.3.3.3]

19004

19006

19005

19007

Attributes:

Binding SID: 119011

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3232, End-point: 8.8.8.8

Name: srte\_c\_3232\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:12:33 (since Apr 14 00:12:52.665)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: EXP\_COLOR-3232\_R8

Requested BSID: dynamic

Explicit: segment-list R4-R3-R5-R6-R8 (valid)

Weight: 1, Metric Type: TE

19004 [Prefix-SID, 4.4.4.4]

19003

19005

19006

19008

Attributes:

Binding SID: 119020

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

On R1 & R2, display cef for CE2 Loopback32 (150.22.2.2) and CE3 Loopback32 (150.23.2.2).

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.22.2.2**

![Shape2](RackMultipart20230222-1-cgwn2a_html_375de1d559d53e15.gif)

Local-label is the binding sid

150.22.2.2/32, version 51, internal 0x5000001 0x30 (ptr 0xd7affa0) [1], 0x0 (0xe1dc848), 0xa08 (0xec01ac8)

Updated May 7 04:43:20.835

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119008, 3 dependencies, recursive [flags 0x6000]

![Shape3](RackMultipart20230222-1-cgwn2a_html_fb07b8123d04d809.gif)

119013 label imposed is the vpn label.

path-idx 0 NHID 0x0 [0xd8f8670 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119008/0/21

next hop srte\_c\_3222\_ labels imposed {ImplNull 119013}

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.2.2**

150.23.2.2/32, version 53, internal 0x5000001 0x30 (ptr 0xd7ae0f0) [1], 0x0 (0xe1dd148), 0xa08 (0xec01a80)

Updated May 7 04:43:20.835

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119011, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f8378 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119011/0/21

next hop srte\_c\_3232\_ labels imposed {ImplNull 119004}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119008**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119008 Pop No ID srte\_c\_3222\_ point2point 0

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119011**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119011 Pop No ID srte\_c\_3232\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | For the first scenario, we will link everything together from the outputs to verify the label stack. Even though this is only shown here, the same steps can be followed for each scenario. |

Before looking at the traceroute let's predict the label stack for traffic hitting the SR-TE policy, entering R1.

For the destination 150.22.2.2, in CEF we see the VPN label is 119013. That will be the last label in the stack.

Also in CEF, we see the binding SID as 119008, and looking the at the SR-TE policy for that B-SID there is an imposed label stack of 19011 / 19003 / 19004.

Since PCE1 (label 19011) is the next hop from R1, it will PHP the top lab and the label stack should be 19003 / 19004 / 119013.

Let's see if that is what we see.

On CE1, traceroute to CE2 Loopback32 (150.22.2.2) & CE3 Loopback32 (150.23.2.2) to display the path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.22.2.2 source lo31 probe 1**

Tue May 17 17:59:31.145 UTC

Type escape sequence to abort.

Tracing the route to 150.22.2.2

1 r1 (172.1.21.0) 0 msec

2 pce1 (172.1.11.1) [MPLS: Labels 19003/19004/119013 Exp 0] 0 msec

3 r3 (172.3.11.0) [MPLS: Labels 19004/119013 Exp 0] 119 msec

4 r4 (172.3.4.101) [MPLS: Label 119013 Exp 0] 19 msec

5 ce2 (172.4.22.1) 19 msec

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.2.2 source lo32 probe 1**

Tue May 17 18:03:04.340 UTC

Type escape sequence to abort.

Tracing the route to 150.23.2.2

1 r1 (172.1.21.0) 19 msec

2 r3 (172.1.3.1) [MPLS: Labels 19004/19006/19005/19007/119004 Exp 0] 39 msec

3 r4 (172.3.4.101) [MPLS: Labels 19006/19005/19007/119004 Exp 0] 9 msec

4 r6 (172.4.6.1) [MPLS: Labels 19005/19007/119004 Exp 0] 79 msec

5 r5 (172.5.6.0) [MPLS: Labels 19007/119004 Exp 0] 9 msec

6 r7 (172.5.7.1) [MPLS: Label 119004 Exp 0] 0 msec

7 ce3 (172.7.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

# Scenario 4 - Inter-Domain Network Slicing for Low Latency

In this scenario, we will slice our network using a dynamic path. A dynamic path does not use a segment list, but it calculates on the head end the shortest path to the destination using the metric type selected. In this scenario, we will create a dynamic path from CE1 to CE3 Loopback33 (150.23.3.3) using delay as the metric.

![](RackMultipart20230222-1-cgwn2a_html_2f80a5b1f5c00c7d.gif)

## Task 1: Configure color Low Latency Traffic

Similar to what we did for Explicit path, we will assign the color at the tail end of the tunnel.

NOTE: We can only configure one route-policy to any given neighbor in each direction also inline modification of a route-policy is not allowed, therefore we will need to recreate the route-policy to include previous lines and add our new lines.

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3233**

**3233**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Dynamic Path - Latency #####**

**elseif destination in (150.23.3.3) then**

**set extcommunity color COLOR-3233**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | There is no need to apply the route-policy in the BGP of R7 & R8 towards CE3 since that was done in scenario 2 already. |

##

## Task 2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the new color.

In R1 & R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.3.3**

BGP routing table entry for 150.23.3.3/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 40 40

Last Modified: Apr 14 04:27:50.705 for 00:00:03

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119010

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 39

Extended community: Color:3233 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119012

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3233 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. |

##

## Task 3: Configure SRTE policy Low Latency Traffic

Similar to the explicit path, a dynamic path is identified by the head-end, color and tail-end tuple. In this task, we will be configuring at the head-end router the policy with the assigned color and tail-end (end-point) however instead of using an explicit list, we will be using a dynamic calculation based on the lowest latency path.

On R1 & R2 apply the following configuration:

**segment-routing**

**traffic-eng**

**policy DYN-Latency\_COLOR-3233\_R7**

**color 3233 end-point ipv4 7.7.7.7**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type latency**

**!**

**!**

**!**

**!**

**!**

**policy DYN-Latency\_COLOR-3233\_R8**

**color 3233 end-point ipv4 8.8.8.8**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type latency**

**!**

**!**

**!**

**!**

**!**

**!**

**!**

NOTE: For inter-domain SR-TE with multiple IGP without mutual redistribution, it is required to have a PCEP that can collect all the link information from all IGPs, without a PCEP, the head end will not have a complete view of the network and it will be unable to create a valid path that traverses multiple IGP.

## Task 4: Verify Service Path

On R1 & R2 display the BGP prefix.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.3.3**

BGP routing table entry for 150.23.3.3/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 83 83

Last Modified: May 7 05:11:04.425 for 00:03:33

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3233 (bsid:119022) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119005

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 81

Extended community: Color:3233 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3233, up, not-registered, bsid 119022

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3233 (bsid:119024) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119003

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3233 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3233, up, not-registered, bsid 119024

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3233**

SR-TE policy database

---------------------

Color: 3233, End-point: 7.7.7.7

Name: srte\_c\_3233\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:02:10 (since Apr 14 04:49:53.203)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3233\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3233\_R7\_discr\_100

PLSP-ID: 3

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119022

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3233, End-point: 8.8.8.8

Name: srte\_c\_3233\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:02:10 (since Apr 14 04:49:53.203)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3233\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3233\_R8\_discr\_100

PLSP-ID: 4

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119024

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

On R1 & R2, display cef for 150.23.3.3

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.3.3**

150.23.3.3/32, version 59, internal 0x5000001 0x30 (ptr 0xd7ae018) [1], 0x0 (0xe1dd190), 0xa08 (0xec01a38)

Updated May 7 05:11:04.452

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119022, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f79d0 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119022/0/21

next hop srte\_c\_3233\_ labels imposed {ImplNull 119005}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119022**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119022 Pop No ID srte\_c\_3233\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 Loopback33 (150.23.3.3) to display the path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.3.3 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.3.3

1 r1 (172.1.21.0) 0 msec

2 pce1 (172.1.11.1) [MPLS: Labels 19003/19012/19005/19013/19007/119010 Exp 0] 9 msec

3 r3 (172.3.11.0) [MPLS: Labels 19012/19005/19013/19007/119010 Exp 0] 0 msec

4 pce2 (172.3.12.1) [MPLS: Labels 19005/19013/19007/119010 Exp 0] 19 msec

5 r5 (172.5.12.0) [MPLS: Labels 19013/19007/119010 Exp 0] 9 msec

6 pce3 (172.5.13.1) [MPLS: Labels 19007/119010 Exp 0] 9 msec

7 r7 (172.7.13.0) [MPLS: Label 119010 Exp 0] 0 msec

8 ce3 (172.7.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

## Task 5: Change the Latency value of a link

In this task, we will temporarily make changes to the delay of PCE2-R5 and PCE2-R6 to simulate degradation of those links which will trigger a re-optimization of the path.

On PCE2 increase the delay to R5 and R6 from 10ms to 150ms.

**performance-measurement**

**interface GigabitEthernet0/0/0/2**

**delay-measurement**

**advertise-delay 150**

**!**

**!**

**interface GigabitEthernet0/0/0/3**

**delay-measurement**

**advertise-delay 150**

**!**

**!**

**!**

## Task 6: Verify Service Path Change

The new SR-TE path will re-optimize to avoid the high delay links.

![](RackMultipart20230222-1-cgwn2a_html_ed9d4513afc3b5e2.gif)

On R1 & R2 display the SR-TE policy details.

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3233**

SR-TE policy database

---------------------

Color: 3233, End-point: 7.7.7.7

Name: srte\_c\_3233\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:04:50 (since Apr 14 04:49:53.203)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3233\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3233\_R7\_discr\_100

PLSP-ID: 3

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 140

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119022

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3233, End-point: 8.8.8.8

Name: srte\_c\_3233\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:04:50 (since Apr 14 04:49:53.203)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3233\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3233\_R8\_discr\_100

PLSP-ID: 4

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 140

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119024

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
 | Output from R2 is similar, omitted for brevity. |

On CE1, traceroute to CE3 to display the new path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.3.3 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.3.3

1 r1 (172.1.21.0) 9 msec

2 pce1 (172.1.11.1) [MPLS: Labels 19003/19005/19013/19007/119010 Exp 0] 9 msec

3 r3 (172.3.11.0) [MPLS: Labels 19005/19013/19007/119010 Exp 0] 0 msec

4 r5 (172.3.5.1) [MPLS: Labels 19013/19007/119010 Exp 0] 0 msec

5 pce3 (172.5.13.1) [MPLS: Labels 19007/119010 Exp 0] 0 msec

6 r7 (172.7.13.0) [MPLS: Label 119010 Exp 0] 0 msec

7 ce3 (172.7.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

##

## Task 7: Restore the original Latency value

On PCE2 restore the delay to R5 and R6 from 150ms to 10ms & 13ms respectively.

**performance-measurement**

**interface GigabitEthernet0/0/0/2**

**delay-measurement**

**advertise-delay 10**

**!**

**!**

**interface GigabitEthernet0/0/0/3**

**delay-measurement**

**advertise-delay 13**

**!**

**!**

**!**

# Scenario 5 - Inter-Domain Network Slicing for TE-Metric

In this scenario, we will slice our network using a dynamic path with TE metric. We will create a dynamic path from CE1 to CE3 Loopback34 (150.23.4.4) using TE metric.

![](RackMultipart20230222-1-cgwn2a_html_ce8a3c921e9d73e2.gif)

## Task 1: Configure color for TE-metric

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | We can only configure one route-policy to any given neighbor in each direction also inline modification of a route-policy is not allowed, therefore we will need to recreate the route-policy to include previous lines and add our new lines. |

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3234**

**3234**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Dynamic Path - Latency #####**

**elseif destination in (150.23.3.3) then**

**set extcommunity color COLOR-3233**

**##### Dynamic Path - TE #####**

**elseif destination in (150.23.4.4) then**

**set extcommunity color COLOR-3234**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | There is no need to apply the route-policy in R7 & R8 towards CE3 since that was done in scenario 2 already. |

## Task 2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.4.4**

BGP routing table entry for 150.23.4.4/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 46 46

Last Modified: Apr 14 15:07:16.705 for 00:00:51

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119006

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 46

Extended community: Color:3234 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119004

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3234 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

RP/0/RP0/CPU0:R1#

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. |

## Task 3: Configure SRTE policy for TE-metric

The configuration of dynamic path with TE metric is similar to the previous scenario where we used latency, the only difference is changing the metric type from latency to TE. Also, since we are going across multiple IGP without mutual redistribution, again we will be using a PCEP to obtain the dynamic path.

On R1 & R2 apply the following configuration:

**segment-routing**

**traffic-eng**

**policy DYN-TE\_COLOR-3234\_R7**

**color 3234 end-point ipv4 7.7.7.7**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type te**

**!**

**!**

**!**

**!**

**!**

**policy DYN-TE\_COLOR-3234\_R8**

**color 3234 end-point ipv4 8.8.8.8**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type te**

**!**

**!**

**!**

**!**

**!**

**!**

**!**

## Task 4: Verify Service Path

On R1 & R2 display the BGP prefix.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.4.4**

BGP routing table entry for 150.23.4.4/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 188 188

Last Modified: May 9 15:29:15.425 for 00:00:34

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3234 (bsid:119026) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119006

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 186

Extended community: Color:3234 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3234, up, not-registered, bsid 119026

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3234 (bsid:119028) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119004

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3234 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3234, up, not-registered, bsid 119028

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3234**

SR-TE policy database

---------------------

Color: 3234, End-point: 7.7.7.7

Name: srte\_c\_3234\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:09:01 (since Apr 14 15:18:09.818)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3234\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3234\_R7\_discr\_100

PLSP-ID: 5

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 2500

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119026

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3234, End-point: 8.8.8.8

Name: srte\_c\_3234\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:09:01 (since Apr 14 15:18:10.018)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3234\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3234\_R8\_discr\_100

PLSP-ID: 6

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 2500

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19006 [Prefix-SID, 6.6.6.6]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119028

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

On R1 & R2, display cef for 150.23.4.4

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.4.4**

Mon May 9 15:36:54.270 UTC

150.23.4.4/32, version 123, internal 0x5000001 0x30 (ptr 0xd7aec08) [1], 0x0 (0xe1dcf08), 0xa08 (0xec01a80)

Updated May 9 15:29:15.495

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119026, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f7cd0 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119026/0/21

next hop srte\_c\_3234\_ labels imposed {ImplNull 119006}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119026**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119026 Pop No ID srte\_c\_3234\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 to display the path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.4.4 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.4.4

1 r2 (172.2.21.0) 0 msec

2 r4 (172.2.4.1) [MPLS: Labels 19012/19006/19008/119015 Exp 0] 0 msec

3 pce2 (172.4.12.1) [MPLS: Labels 19006/19008/119015 Exp 0] 0 msec

4 r6 (172.6.12.0) [MPLS: Labels 19008/119015 Exp 0] 19 msec

5 r8 (172.6.8.1) [MPLS: Label 119015 Exp 0] 0 msec

6 ce3 (172.8.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. |

## Task 5: Change TE metric of a link

In this task, we will temporarily make changes to the TE metric of R1-R3 and R2-R4 links from 1000 to 5000 to force a re-optimization.

On R1 and R2 issue the following command:

**segment-routing**

**traffic-eng**

**interface GigabitEthernet0/0/0/3**

**metric 5000**

**!**

**!**

**!**

## Task 6: Verify Service Path Change

The new SR-TE path should avoid the high TE metric links

![](RackMultipart20230222-1-cgwn2a_html_7c97783a2de6fb3d.gif)

On R1 & R2 display the SR-TE policy details.

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3234**

SR-TE policy database

---------------------

Color: 3234, End-point: 7.7.7.7

Name: srte\_c\_3234\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 01:43:42 (since Apr 14 15:18:09.818)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3234\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3234\_R7\_discr\_100

PLSP-ID: 5

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 3500

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119026

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3234, End-point: 8.8.8.8

Name: srte\_c\_3234\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 01:43:42 (since Apr 14 15:18:10.018)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: DYN\_COLOR-3234\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_DYN\_COLOR-3234\_R8\_discr\_100

PLSP-ID: 6

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 3500

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19006 [Prefix-SID, 6.6.6.6]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119028

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
 | Output from R2 is similar, omitted for brevity. |

On CE1, traceroute CE3 Loopback 34 to display the new path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.4.4 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.4.4

1 r2 (172.2.21.0) 0 msec

2 pce1 (172.2.11.1) [MPLS: Labels 19003/19012/19006/19008/119015 Exp 0] 0 msec

3 r3 (172.3.11.0) [MPLS: Labels 19012/19006/19008/119015 Exp 0] 0 msec

4 pce2 (172.3.12.1) [MPLS: Labels 19006/19008/119015 Exp 0] 9 msec

5 r6 (172.6.12.0) [MPLS: Labels 19008/119015 Exp 0] 0 msec

6 r8 (172.6.8.1) [MPLS: Label 119015 Exp 0] 0 msec

7 ce3 (172.8.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. |

## Task 7: Restore original TE metric

On R1 and R2 issue the following command:

**segment-routing**

**traffic-eng**

**interface GigabitEthernet0/0/0/3**

**metric 1000**

**!**

**!**

**!**

# Scenario 6 - Inter-Domain Network Slicing for Regular Traffic with Anycast SID

In this scenario, we will configure and use an Anycast SID on R5 and R6. An Anycast SID is a type of prefix SID that identifies a set of nodes and is configured with n-flag clear. The set of nodes (Anycast group) is configured to advertise a shared prefix address and prefix SID. Anycast routing enables the steering of traffic toward multiple advertising nodes, providing load-balancing and redundancy. Packets addressed to an Anycast address are forwarded to the topologically nearest nodes.

![](RackMultipart20230222-1-cgwn2a_html_1ce536dd7f6b05fb.gif)

## Task 1: Configure color for Anycast SID

For this scenario we will be using CE3 Loopback 35 (150.23.5.5) with color 3235, like previous scenarios we need to create the extended community and update the route-policy.

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3235**

**3235**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Dynamic Path - Latency #####**

**elseif destination in (150.23.3.3) then**

**set extcommunity color COLOR-3233**

**##### Dynamic Path - TE #####**

**elseif destination in (150.23.4.4) then**

**set extcommunity color COLOR-3234**

**##### Anycast SID - TE #####**

**elseif destination in (150.23.5.5) then**

**set extcommunity color COLOR-3235**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | There is no need to apply the route-policy in R7 & R8 towards CE3 since that was done in scenario 2 already. |

## Task 2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.5.5**

BGP routing table entry for 150.23.5.5/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 58 58

Last Modified: Apr 19 04:13:41.272 for 00:03:37

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119007

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 57

Extended community: Color:3235 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119005

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3235 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. |

## Task 3: Configure Anycast-SID at ABRs

In R5 and R6, we will need to create a new Loopback interface and advertise it via IS-IS with the n-flag-clear to be used as an Anycast SID.

Apply the following configuration in R5 and R6:

**interface Loopback56**

**ipv4 address 56.56.56.56 255.255.255.255**

**!**

**router isis ACCESS-2**

**interface Loopback56**

**address-family ipv4 unicast**

**tag 100**

**prefix-sid index 56 n-flag-clear**

**!**

**!**

**!**

**router isis AGG-CORE**

**interface Loopback56**

**address-family ipv4 unicast**

**tag 100**

**prefix-sid index 56 n-flag-clear**

**!**

**!**

**!**

## Task 4: Configure SR-TE Policy using anycast SID

On R1 & R2 apply the following configuration:

**segment-routing**

**traffic-eng**

**policy ANYCAST-IGP\_COLOR-3235\_R7**

**color 3235 end-point ipv4 7.7.7.7**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type igp**

**!**

**anycast-sid-inclusion**

**!**

**!**

**!**

**!**

**!**

**policy ANYCAST-IGP\_COLOR-3235\_R8**

**color 3235 end-point ipv4 8.8.8.8**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type igp**

**!**

**anycast-sid-inclusion**

**!**

**!**

**!**

**!**

**!**

**!**

**!**

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | For the Anycast SID to be used in the path calculation we must add the anycast-sid-inclusion CLI under the policy. |

## Task 5: Verify Service Path

On R1 & R2 display the BGP prefix.

RP/0/RP0/CPU0:R1#sh bgp vrf CUSTOMER-A 150.23.5.5

BGP routing table entry for 150.23.5.5/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 191 191

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3235 (bsid:119034) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119007

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 189

Extended community: Color:3235 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3235, up, not-registered, bsid 119034

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3235 (bsid:119036) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119005

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3235 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3235, up, not-registered, bsid 119036

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 and R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3235**

SR-TE policy database

---------------------

Color: 3235, End-point: 7.7.7.7

Name: srte\_c\_3235\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 1w1d (since May 9 15:46:10.411)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: ANYCAST-IGP\_COLOR-3235\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_ANYCAST-IGP\_COLOR-3235\_R7\_discr\_100

PLSP-ID: 5

Maximum SID Depth: 10

Anycast Inclusion: Enabled

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 300

19003 [Prefix-SID, 3.3.3.3]

19056 [Prefix-SID, 56.56.56.56]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119034

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3235, End-point: 8.8.8.8

Name: srte\_c\_3235\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 1w1d (since May 9 15:46:10.823)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: ANYCAST-IGP\_COLOR-3235\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_ANYCAST-IGP\_COLOR-3235\_R8\_discr\_100

PLSP-ID: 6

Maximum SID Depth: 10

Anycast Inclusion: Enabled

Dynamic (pce 11.11.11.11) (valid)

Metric Type: IGP, Path Accumulated Metric: 400

19003 [Prefix-SID, 3.3.3.3]

19006 [Prefix-SID, 6.6.6.6]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119036

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

On R1 & R2, display cef for 150.23.5.5

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.5.5**

150.23.5.5/32, version 129, internal 0x5000001 0x30 (ptr 0xd7aeb30) [1], 0x0 (0xe1dcec0), 0xa08 (0xec018d0)

Updated May 9 15:46:12.922

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119034, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f6c70 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119034/0/21

next hop srte\_c\_3235\_ labels imposed {ImplNull 119007}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119034**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119034 Pop No ID srte\_c\_3235\_ point2point 160

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 to display the path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.5.5 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.5.5

1 r1 (172.1.21.0) 9 msec

2 r3 (172.1.3.1) [MPLS: Labels 19056/19007/119007 Exp 0] 0 msec

3 r5 (172.3.5.1) [MPLS: Labels 19007/119007 Exp 0] 0 msec

4 r7 (172.5.7.1) [MPLS: Label 119007 Exp 0] 0 msec

5 ce3 (172.7.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

# Scenario 7 - Inter-Domain Network Slicing with link affinities

Affinity constraints allow the head-end router to compute a dynamic path that includes or excludes links that have specific colors or combinations of colors. We will create a dynamic path from CE1 to CE3 Loopback36 (150.23.6.6) using TE-metric and excluding RED links.

![](RackMultipart20230222-1-cgwn2a_html_ab12831a4238472e.gif)

## Task 1: Configure color

For this scenario, we will be using CE3 150.23.6.6 with color 3236, similar to previous scenarios we need to create the extended community and update the route-policy.

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3236**

**3236**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Dynamic Path - Latency #####**

**elseif destination in (150.23.3.3) then**

**set extcommunity color COLOR-3233**

**##### Dynamic Path - TE #####**

**elseif destination in (150.23.4.4) then**

**set extcommunity color COLOR-3234**

**##### Anycast SID - TE #####**

**elseif destination in (150.23.5.5) then**

**set extcommunity color COLOR-3235**

**##### Affinity - TE #####**

**elseif destination in (150.23.6.6) then**

**set extcommunity color COLOR-3236**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | There is no need to apply the route-policy in R7 & R8 towards CE3 since that was done in scenario 2 already. |

## Task 2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following command:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.6.6**

Tue Apr 19 04:50:57.600 UTC

BGP routing table entry for 150.23.6.6/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 60 60

Last Modified: Apr 19 04:50:32.272 for 00:00:25

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119008

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 59

Extended community: Color:3236 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119006

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3236 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. |

##

## Task 3: Configure color to avoid using links with marked affinity (red link)

To use the affinity constraints, we will need to color the links that we want to reference to exclude or include in our path, we achieve this by adding a name to the links and then providing an ID to that name by using bit-position.

On R1 and R2 issue the following command:

**segment-routing**

**traffic-eng**

**interface GigabitEthernet0/0/0/3**

**affinity**

**name RED**

**!**

**!**

**affinity-map**

**name RED bit-position 1**

**!**

**!**

**!**

## Task 4: Configure SRTE policy to avoid red links

On R1 and R2 issue the following command:

**segment-routing**

**traffic-eng**

**policy AFF-TE\_COLOR-3236\_R7**

**color 3236 end-point ipv4 7.7.7.7**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type te**

**!**

**!**

**constraints**

**affinity**

**exclude-any**

**name RED**

**!**

**!**

**!**

**root**

**segment-routing**

**traffic-eng**

**policy AFF-TE\_COLOR-3236\_R8**

**color 3236 end-point ipv4 8.8.8.8**

**candidate-paths**

**preference 100**

**dynamic**

**pcep**

**!**

**metric**

**type te**

**!**

**!**

**constraints**

**affinity**

**exclude-any**

**name RED**

**!**

**!**

**!**

**!**

**!**

**!**

**!**

**!**

## Task 5: Verify service Path

On R1 & R2 display the BGP prefix.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.6.6**

BGP routing table entry for 150.23.6.6/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 194 194

Last Modified: May 9 15:57:47.425 for 00:01:40

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3236 (bsid:119038) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119008

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 192

Extended community: Color:3236 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3236, up, not-registered, bsid 119038

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3236 (bsid:119040) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119006

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3236 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3236, up, not-registered, bsid 119040

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3236**

SR-TE policy database

---------------------

Color: 3236, End-point: 7.7.7.7

Name: srte\_c\_3236\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:03:11 (since May 9 15:57:44.943)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: AFF-TE\_COLOR-3236\_R7

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_AFF-TE\_COLOR-3236\_R7\_discr\_100

PLSP-ID: 7

Constraints:

Affinity:

exclude-any:

RED

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 3500

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119038

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3236, End-point: 8.8.8.8

Name: srte\_c\_3236\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:03:11 (since May 9 15:57:45.158)

Candidate-paths:

Preference: 100 (configuration) (active)

Name: AFF-TE\_COLOR-3236\_R8

Requested BSID: dynamic

PCC info:

Symbolic name: cfg\_AFF-TE\_COLOR-3236\_R8\_discr\_100

PLSP-ID: 8

Constraints:

Affinity:

exclude-any:

RED

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: TE, Path Accumulated Metric: 3500

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19006 [Prefix-SID, 6.6.6.6]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119040

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

On R1 & R2, display cef for CE3 Loopback 36 (150.23.6.6).

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.6.6**

Mon May 9 16:03:00.379 UTC

150.23.6.6/32, version 135, internal 0x5000001 0x30 (ptr 0xd7aea58) [1], 0x0 (0xe1dce78), 0xa08 (0xec01720)

Updated May 9 15:57:47.653

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119038, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f6340 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119038/0/21

next hop srte\_c\_3236\_ labels imposed {ImplNull 119008}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info.

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119038**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119038 Pop No ID srte\_c\_3236\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 to display the new path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.6.6 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.6.6

1 r1 (172.1.21.0) 0 msec

2 pce1 (172.1.11.1) [MPLS: Labels 19003/19012/19005/19007/119021 Exp 0] 0 msec

3 r3 (172.3.11.0) [MPLS: Labels 19012/19005/19007/119021 Exp 0] 0 msec

4 pce2 (172.3.12.1) [MPLS: Labels 19005/19007/119021 Exp 0] 0 msec

5 r5 (172.5.12.0) [MPLS: Labels 19007/119021 Exp 0] 0 msec

6 r7 (172.5.7.1) [MPLS: Label 119021 Exp 0] 0 msec

7 ce3 (172.7.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

# Scenario 8 – Intra and Inter-Domain Network Slicing for On-Demand Next Hop (ODN)

Segment Routing On-Demand Next Hop (ODN) allows a service head-end router to automatically instantiate an SR policy to a BGP next-hop when required (on-demand).

For this scenario, we will be using CE2 Loopback 37 (150.22.7.7) and CE3 Loopback37 (150.23.7.7) both will use the color 3207

![](RackMultipart20230222-1-cgwn2a_html_a9bf7dbf27925959.gif)

## Task 1: Configure color for ODN

Create the extended community and update the route-policy.

Apply the following configuration in R4:

**extcommunity-set opaque COLOR-3207**

**3207**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3222 #####**

**if destination in (150.22.2.2) then**

**set extcommunity color COLOR-3222**

**##### ODN - Latency #####**

**elseif destination in (150.22.7.7) then**

**set extcommunity color COLOR-3207**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

Apply the following configuration in R7 and R8:

**extcommunity-set opaque COLOR-3207**

**3207**

**end-set**

**!**

**route-policy CUST-A\_SET\_COLOR\_IN**

**##### Explicit Path - Color 3232 #####**

**if destination in (150.23.2.2) then**

**set extcommunity color COLOR-3232**

**##### Dynamic Path - Latency #####**

**elseif destination in (150.23.3.3) then**

**set extcommunity color COLOR-3233**

**##### Dynamic Path - TE #####**

**elseif destination in (150.23.4.4) then**

**set extcommunity color COLOR-3234**

**##### Anycast SID - TE #####**

**elseif destination in (150.23.5.5) then**

**set extcommunity color COLOR-3235**

**##### Affinity - TE #####**

**elseif destination in (150.23.6.6) then**

**set extcommunity color COLOR-3236**

**##### ODN - Latency #####**

**elseif destination in (150.23.7.7) then**

**set extcommunity color COLOR-3207**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

NOTE: There is no need to apply the route-policy in R4, R7 & R8 towards CE3 since that was done in scenario 2 already.

## Task 2: Verify the prefix is tagged with the new color

In our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color.

In R1 & R2 issue the following commands:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.22.7.7**

BGP routing table entry for 150.22.7.7/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 197 197

Last Modified: May 9 16:10:51.425 for 00:01:25

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

4.4.4.4 (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119016

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 197

Extended community: Color:3207 RT:65001:3

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:3

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.7.7**

BGP routing table entry for 150.23.7.7/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 63 63

Last Modified: Apr 19 04:57:32.272 for 00:00:15

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119009

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 62

Extended community: Color:3207 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119007

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3207 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. |

## Task 3: Configure SRTE policy to use ODN vs defined end point.

The configuration of SR-TE ODN does not use the end-point IP, instead it dynamically creates the path based on the received BGP routes next hop, then the head-end will contact the PCE and request the path towards the destination that minimize the cumulative latency

On R1 and R2 issue the following command:

**segment-routing**

**traffic-eng**

**on-demand color 3207**

**dynamic**

**pcep**

**!**

**metric**

**type latency**

**!**

**!**

**!**

**!**

**!**

## Task 4: Verify Service Path

On R1 & R2 display the BGP prefixes.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.22.7.7**

BGP routing table entry for 150.22.7.7/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 198 198

Last Modified: May 9 16:14:18.425 for 00:00:30

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

4.4.4.4 C:3207 (bsid:119042) (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119016

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 197

Extended community: Color:3207 RT:65001:3

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

SR policy color 3207, up, registered, bsid 119042, if-handle 0x0000009c

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:3

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-A 150.23.7.7**

BGP routing table entry for 150.23.7.7/32, Route Distinguisher: 1.1.1.1:3

Versions:

Process bRIB/RIB SendTblVer

Speaker 199 199

Last Modified: May 9 16:14:18.425 for 00:01:32

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.1

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.1

Local

7.7.7.7 C:3207 (bsid:119043) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119009

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 195

Extended community: Color:3207 RT:65001:3

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3207, up, registered, bsid 119043, if-handle 0x000000a4

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:3

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:3207 (bsid:119044) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119007

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:3207 RT:65001:3

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 3207, up, registered, bsid 119044, if-handle 0x000000ac

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:3

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 3207**

SR-TE policy database

---------------------

Color: 3207, End-point: 4.4.4.4

Name: srte\_c\_3207\_ep\_4.4.4.4

Status:

Admin: up Operational: up for 00:00:15 (since Apr 20 03:28:32.771)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: LATENCY, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_3207\_ep\_4.4.4.4\_discr\_100

PLSP-ID: 12

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 23

19011 [Prefix-SID, 11.11.11.11]

19004 [Prefix-SID, 4.4.4.4]

Attributes:

Binding SID: 119042

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3207, End-point: 7.7.7.7

Name: srte\_c\_3207\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:00:15 (since Apr 20 03:28:32.780)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: LATENCY, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_3207\_ep\_7.7.7.7\_discr\_100

PLSP-ID: 13

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19007 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119043

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 3207, End-point: 8.8.8.8

Name: srte\_c\_3207\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:00:15 (since Apr 20 03:28:32.781)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: LATENCY, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_3207\_ep\_8.8.8.8\_discr\_100

PLSP-ID: 14

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19008 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119044

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

On R1 & R2, display cef for 150.22.7.7 and 150.23.7.7

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.22.7.7**

150.22.7.7/32, version 142, internal 0x5000001 0x30 (ptr 0xd7ad288) [1], 0x0 (0xe1ddf10), 0xa08 (0xec02260)

Updated May 9 16:14:18.789

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119042, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f5840 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119042/0/21

next hop srte\_c\_3207\_ labels imposed {ImplNull 119016}

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-A 150.23.7.7**

150.23.7.7/32, version 144, internal 0x5000001 0x30 (ptr 0xd7ae890) [1], 0x0 (0xe1dd148), 0xa08 (0xec022a8)

Updated May 9 16:14:18.790

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119043, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f5690 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119043/0/21

next hop srte\_c\_3207\_ labels imposed {ImplNull 119009}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119042**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119042 Pop No ID srte\_c\_3207\_ point2point 0

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119043**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119043 Pop No ID srte\_c\_3207\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE2 and CE3 to display the new path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.22.7.7 probe 1**

Type escape sequence to abort.

Tracing the route to 150.22.7.7

1 r2 (172.2.21.0) 9 msec

2 pce1 (172.2.11.1) [MPLS: Labels 19004/119015 Exp 0] 0 msec

3 r4 (172.4.11.0) [MPLS: Label 119015 Exp 0] 0 msec

4 ce2 (172.4.22.1) 0 msec

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-A 150.23.7.7 probe 1**

Wed Apr 20 03:30:29.732 UTC

Type escape sequence to abort.

Tracing the route to 150.23.7.7

1 r2 (172.2.21.0) 9 msec

2 pce1 (172.2.11.1) [MPLS: Labels 19003/19012/19005/19013/19008/119011 Exp 0] 0 msec

3 r3 (172.3.11.0) [MPLS: Labels 19012/19005/19013/19008/119011 Exp 0] 9 msec

4 pce2 (172.3.12.1) [MPLS: Labels 19005/19013/19008/119011 Exp 0] 9 msec

5 r5 (172.5.12.0) [MPLS: Labels 19013/19008/119011 Exp 0] 9 msec

6 pce3 (172.5.13.1) [MPLS: Labels 19008/119011 Exp 0] 9 msec

7 r8 (172.8.13.0) [MPLS: Label 119011 Exp 0] 9 msec

8 ce3 (172.8.23.1) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed R1 instead, output using R1 is omitted for brevity. |

# Scenario 9 - Configure Flex Algo & ODN for low latency

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

![](RackMultipart20230222-1-cgwn2a_html_69c815519cf25695.gif)

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

##

## Task 1: Configure Flex Algo 128 for low latency and a prefix SID on the ABR Nodes

Configure the below commands on R3, R4, R5 & R6:

Apply the following configuration in R3:

**router isis ACCESS-1**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 103128**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 103128**

**!**

**!**

Apply the following configuration in R4:

**router isis ACCESS-1**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 104128**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 104128**

**!**

**!**

Apply the following configuration in R5:

**router isis ACCESS-2**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 105128**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 105128**

**!**

**!**

Apply the following configuration in R6:

**router isis ACCESS-2**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 106128**

**!**

**!**

**!**

**router isis AGG-CORE**

**flex-algo 128**

**metric-type delay**

**advertise-definition**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 106128**

**!**

**!**

## Task 2: Configure Flex Algo 128 for low latency and a prefix SID on all the remaining ISIS nodes

Configure the below commands on R1, R2, PCE1, PCE2, PCE3, R7 & R8

Apply the following configuration in R1:

**router isis ACCESS-1**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 101128**

**!**

**!**

Apply the following configuration in R2:

**router isis ACCESS-1**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 102128**

**!**

**!**

Apply the following configuration in PCE1:

**router isis ACCESS-1**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 111128**

**!**

**!**

Apply the following configuration in PCE2:

**router isis AGG-CORE**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 112128**

**!**

**!**

Apply the following configuration in PCE3:

**router isis ACCESS-2**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 113128**

**!**

**!**

Apply the following configuration in R7:

**router isis ACCESS-2**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 107128**

**!**

**!**

Apply the following configuration in R8:

**router isis ACCESS-2**

**flex-algo 128**

**!**

**interface Loopback0**

**address-family ipv4 unicast**

**prefix-sid algorithm 128 absolute 108128**

**!**

**!**

## Task 3: Verify Flex Algo 128 on all ISIS nodes

Issue the following command on R1, R2 & PCE1:

RP/0/RP0/CPU0:R1# **sh isis flex-algo 128**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R4.00

Definition Equal to Local: No

Definition Metric Type: Delay

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
 | Output from R2 & PCE1 are similar, omitted for brevity. |

Issue the following command on R3, R4 & PCE2:

RP/0/RP0/CPU0:R3# **sh isis flex-algo 128**

IS-IS ACCESS-1 Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R4.00

Definition Equal to Local: Yes

Definition Metric Type: Delay

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

IS-IS AGG-CORE Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: Yes

Definition Metric Type: Delay

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
 | Output from R4 & PCE2 are similar, omitted for brevity. |

Issue the following commands on R5, R6, PCE3, R7 & R8:

RP/0/RP0/CPU0:R5# **show isis flex-algo 128**

IS-IS ACCESS-2 Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: Yes

Definition Metric Type: Delay

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

IS-IS AGG-CORE Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: Yes

Definition Metric Type: Delay

Definition Flex-Algo Prefix Metric: No

Disabled: No

Local Priority: 128

FRR Disabled: No

Microloop Avoidance Disabled: No

RP/0/RP0/CPU0:R7# **sh isis flex-algo 128**

Tue May 3 02:29:09.983 UTC

IS-IS ACCESS-2 Flex-Algo Database

Flex-Algo 128:

Level-1:

Definition Priority: 128

Definition Source: R6.00

Definition Equal to Local: No

Definition Metric Type: Delay

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
 | Output from R6,PCE3 & R8 are similar, omitted for brevity. |

## Task 4: RPL to Color CE prefixes of VRF CUSTOMER-B on R4, R7 & R8

Apply the following configuration in R4:

**extcommunity-set opaque COLOR-4128**

**4128**

**end-set**

**!**

**route-policy CUST-B\_SET\_COLOR\_IN**

**##### ODN Flex Algo 128 - Latency #####**

**if destination in (150.22.7.7) then**

**set extcommunity color COLOR-4128**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

**router bgp 65001**

**vrf CUSTOMER-B**

**neighbor 172.4.22.101**

**address-family ipv4 unicast**

**route-policy CUST-B\_SET\_COLOR\_IN in**

**!**

**!**

**!**

**!**

Apply the following configuration in R7:

**extcommunity-set opaque COLOR-4128**

**4128**

**end-set**

**!**

**route-policy CUST-B\_SET\_COLOR\_IN**

**##### ODN Flex Algo 128 - Latency #####**

**if destination in (150.23.7.7) then**

**set extcommunity color COLOR-4128**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

**router bgp 65001**

**vrf CUSTOMER-B**

**neighbor 172.7.23.101**

**address-family ipv4 unicast**

**route-policy CUST-B\_SET\_COLOR\_IN in**

**!**

**!**

**!**

**!**

Apply the following configuration in R8:

**extcommunity-set opaque COLOR-4128**

**4128**

**end-set**

**!**

**route-policy CUST-B\_SET\_COLOR\_IN**

**##### ODN Flex Algo 128 - Latency #####**

**if destination in (150.23.7.7) then**

**set extcommunity color COLOR-4128**

**##### Everything Else #####**

**else**

**pass**

**endif**

**end-policy**

**!**

**router bgp 65001**

**vrf CUSTOMER-B**

**neighbor 172.8.23.101**

**address-family ipv4 unicast**

**route-policy CUST-B\_SET\_COLOR\_IN in**

**!**

**!**

**!**

**!**

## Task 5: Verify the prefix is tagged with the new color

On our path head-end (R1 & R2) we will display the BGP attributes of our prefix to make sure it has been tagged with the right color (4128).

On R1 & R2 issue the following commands:

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.22.7.7**

BGP routing table entry for 150.22.7.7/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 122 122

Last Modified: May 3 03:03:40.059 for 00:00:44

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

4.4.4.4 (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119015

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 122

Extended community: **Color:4128** RT:65001:4

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:4

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.23.7.7**

BGP routing table entry for 150.23.7.7/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 117 117

Last Modified: May 3 02:44:33.059 for 00:15:44

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

7.7.7.7 (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119018

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 116

Extended community: **Color:4128** RT:65001:4

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:4

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119007

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: **Color:4128** RT:65001:4

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:4

NOTE: Output from R2 is similar, omitted for brevity.

## Task 6: ODN and flex Algo 128 policy on R1 & R2

Configure SR Policy with ODN and Flex Algo on the head end R1 and R2.

Apply the following configuration in R1 & R2:

**segment-routing**

**traffic-eng**

**on-demand color 4128**

**dynamic**

**pcep**

**!**

**sid-algorithm 128**

**!**

**!**

**!**

**!**

## Task 7: Verify Service Path

On R1 & R2 display the BGP prefixes.

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.22.7.7**

BGP routing table entry for 150.22.7.7/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 203 203

Last Modified: May 9 21:34:15.425 for 02:36:05

Paths: (1 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

4.4.4.4 C:4128 (bsid:119072) (metric 300) from 11.11.11.11 (4.4.4.4)

Received Label 119004

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 202

Extended community: Color:4128 RT:65001:4

Originator: 4.4.4.4, Cluster list: 11.11.11.11

EVPN Gateway Address : 0.0.0.0

SR policy color 4128, up, registered, bsid 119072, if-handle 0x000000b4

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 4.4.4.4:4

RP/0/RP0/CPU0:R1# **sh bgp vrf CUSTOMER-B 150.23.7.7**

BGP routing table entry for 150.23.7.7/32, Route Distinguisher: 1.1.1.1:4

Versions:

Process bRIB/RIB SendTblVer

Speaker 204 204

Last Modified: May 9 21:34:15.425 for 02:36:13

Paths: (2 available, best #1)

Advertised to CE peers (in unique update groups):

172.1.21.101

Path #1: Received by speaker 0

Advertised to CE peers (in unique update groups):

172.1.21.101

Local

7.7.7.7 C:4128 (bsid:119075) (metric 600) from 11.11.11.11 (7.7.7.7)

Received Label 119018

Origin incomplete, metric 0, localpref 100, valid, internal, best, group-best, import-candidate, imported

Received Path ID 1, Local Path ID 1, version 200

Extended community: Color:4128 RT:65001:4

Originator: 7.7.7.7, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4128, up, registered, bsid 119075, if-handle 0x000000bc

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 7.7.7.7:4

Path #2: Received by speaker 0

Not advertised to any peer

Local

8.8.8.8 C:4128 (bsid:119076) (metric 700) from 11.11.11.11 (8.8.8.8)

Received Label 119020

Origin incomplete, metric 0, localpref 100, valid, internal, import-candidate, imported

Received Path ID 1, Local Path ID 0, version 0

Extended community: Color:4128 RT:65001:4

Originator: 8.8.8.8, Cluster list: 11.11.11.11, 3.3.3.3, 12.12.12.12, 5.5.5.5, 13.13.13.13

EVPN Gateway Address : 0.0.0.0

SR policy color 4128, up, registered, bsid 119076, if-handle 0x000000c4

Source AFI: L2VPN EVPN, Source VRF: default, Source Route Distinguisher: 8.8.8.8:4

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab |

On R1 & R2 display the SR-TE policy details then observe the path and verify that the Binding SID matches the previous output

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 4128**

SR-TE policy database

---------------------

Color: 4128, End-point: 4.4.4.4

Name: srte\_c\_4128\_ep\_4.4.4.4

Status:

Admin: up Operational: up for 00:07:30 (since May 3 03:17:04.171)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4128\_ep\_4.4.4.4\_discr\_100

PLSP-ID: 12

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 23

104128 [Prefix-SID, 4.4.4.4]

Attributes:

Binding SID: 119072

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 4128, End-point: 7.7.7.7

Name: srte\_c\_4128\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:07:30 (since May 3 03:17:04.205)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4128\_ep\_7.7.7.7\_discr\_100

PLSP-ID: 13

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

107128 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119075

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 4128, End-point: 8.8.8.8

Name: srte\_c\_4128\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:07:30 (since May 3 03:17:04.205)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4128\_ep\_8.8.8.8\_discr\_100

PLSP-ID: 14

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 60

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

108128 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119076

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

On R1 & R2, display cef for 150.22.7.7 and 150.23.7.7

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-B 150.22.7.7**

150.22.7.7/32, version 130, internal 0x5000001 0x30 (ptr 0xd7af960) [1], 0x0 (0xe1dc920), 0xa08 (0xec026e0)

Updated May 9 21:34:15.270

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119072, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f6668 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119072/0/21

next hop srte\_c\_4128\_ labels imposed {ImplNull 119004}

RP/0/RP0/CPU0:R1# **sh cef vrf CUSTOMER-B 150.23.7.7**

150.23.7.7/32, version 132, internal 0x5000001 0x30 (ptr 0xd7adb10) [1], 0x0 (0xe1dd070), 0xa08 (0xec02410)

Updated May 9 21:34:15.270

Prefix Len 32, traffic index 0, precedence n/a, priority 3

via local-label 119075, 3 dependencies, recursive [flags 0x6000]

path-idx 0 NHID 0x0 [0xd8f6ac0 0x0]

recursion-via-label

next hop VRF - 'default', table - 0xe0000000

next hop via 119075/0/21

next hop srte\_c\_4128\_ labels imposed {ImplNull 119018}

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On R1 & R2, display the binding SID info

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119072**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119072 Pop No ID srte\_c\_4128\_ point2point 0

RP/0/RP0/CPU0:R1# **sh mpls forwarding labels 119075**

Local Outgoing Prefix Outgoing Next Hop Bytes

Label Label or ID Interface Switched

------ ----------- ------------------ ------------ --------------- ------------

119075 Pop No ID srte\_c\_4128\_ point2point 0

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Output from R2 is similar, omitted for brevity. Binding SID may have a different value in your lab. |

On CE1, traceroute to CE3 to display the path taken.

Below traceroute follow **inter domain path** using labels from ODN and Flex algo 128 policy on R1 and R2 covered in previous steps

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-B 150.23.7.7 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.7.7

1 r2 (172.2.21.100) 0 msec

2 pce1 (172.2.11.1) [MPLS: Labels 103128/105128/108128/119007 Exp 0] 0 msec

3 r3 (172.3.11.0) [MPLS: Labels 105128/108128/119007 Exp 0] 0 msec

4 pce2 (172.3.12.1) [MPLS: Labels 105128/108128/119007 Exp 0] 249 msec

5 r5 (172.5.12.0) [MPLS: Labels 108128/119007 Exp 0] 0 msec

6 pce3 (172.5.13.1) [MPLS: Labels 108128/119007 Exp 0] 0 msec

7 r8 (172.8.13.0) [MPLS: Label 119007 Exp 0] 0 msec

8 ce3 (172.8.23.101) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. |

##

Below traceroute follow **intra domain path using** labels fromODN and Flex algo 128 policy on R1 and R2 covered in previous steps.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-B 150.22.7.7 probe 1**

Type escape sequence to abort.

Tracing the route to 150.22.7.7

1 r2 (172.2.21.100) 0 msec

2 pce1 (172.2.11.1) [MPLS: Labels 104128/119015 Exp 0] 0 msec

3 r4 (172.4.11.0) [MPLS: Label 119015 Exp 0] 0 msec

4 ce2 (172.4.22.101) 0 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R1 instead, output using R1 is omitted for brevity. |

Prefix-SIDs of Flex algorithm allows SRTE paths to be expressed in a better way by reducing the number of SID's required in the SID list.

Below Section compares the SID LIST of ODN SR Policy (Color 3207) and Flex Algo + ODN SR policy (Color 4128).

RP/0/RP0/CPU0:R1# **show segment-routing traffic-eng policy color 3207 endpoint ipv4 7.7.7.7 | in Prefix-SID**

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19007 [Prefix-SID, 7.7.7.7]

RP/0/RP0/CPU0:R1# **show segment-routing traffic-eng policy color 4128 endpoint ipv4 7.7.7.7 | in Prefix-SID**

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

107128 [Prefix-SID, 7.7.7.7]

RP/0/RP0/CPU0:R1# **show segment-routing traffic-eng policy color 3207 endpoint ipv4 8.8.8.8 | in Prefix-SID**

19011 [Prefix-SID, 11.11.11.11]

19003 [Prefix-SID, 3.3.3.3]

19012 [Prefix-SID, 12.12.12.12]

19005 [Prefix-SID, 5.5.5.5]

19013 [Prefix-SID, 13.13.13.13]

19008 [Prefix-SID, 8.8.8.8]

RP/0/RP0/CPU0:R1# **show segment-routing traffic-eng policy color 4128 endpoint ipv4 8.8.8.8 | in Prefix-SID**

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

108128 [Prefix-SID, 8.8.8.8]

##

## Task 8: Change Latency value of a link

In this task, we will temporarily make changes to the delay of PCE2-R5 and PCE2-R6 to simulate a degradation of those links which will trigger a re-optimization of the path.

On PCE2 configure the delay to R5 and R6 as 150ms.

**performance-measurement**

**interface GigabitEthernet0/0/0/2**

**delay-measurement**

**advertise-delay 150**

**!**

**!**

**interface GigabitEthernet0/0/0/3**

**delay-measurement**

**advertise-delay 150**

**!**

**!**

**!**

##

## Task 9: Verify Service Path Change

The new SR-TE path should re-optimize to avoid the high delay links.

![](RackMultipart20230222-1-cgwn2a_html_65dd843e42a2b8e.gif)

On R1 & R2 display the SR-TE policy details.

RP/0/RP0/CPU0:R1# **sh segment-routing traffic-eng policy color 4128**

SR-TE policy database

---------------------

Color: 4128, End-point: 7.7.7.7

Name: srte\_c\_4128\_ep\_7.7.7.7

Status:

Admin: up Operational: up for 00:38:19 (since May 3 03:17:04.205)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4128\_ep\_7.7.7.7\_discr\_100

PLSP-ID: 13

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 140

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

107128 [Prefix-SID, 7.7.7.7]

Attributes:

Binding SID: 119075

Forward Class: Not Configured

Steering labeled-services disabled: no

Steering BGP disabled: no

IPv6 caps enable: yes

Color: 4128, End-point: 8.8.8.8

Name: srte\_c\_4128\_ep\_8.8.8.8

Status:

Admin: up Operational: up for 00:38:19 (since May 3 03:17:04.205)

Candidate-paths:

Preference: 200 (BGP ODN) (shutdown)

Requested BSID: dynamic

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (invalid)

Metric Type: TE, Path Accumulated Metric: 0

Preference: 100 (BGP ODN) (active)

Requested BSID: dynamic

PCC info:

Symbolic name: bgp\_c\_4128\_ep\_8.8.8.8\_discr\_100

PLSP-ID: 14

Constraints:

Prefix-SID Algorithm: 128

Maximum SID Depth: 10

Dynamic (pce 11.11.11.11) (valid)

Metric Type: LATENCY, Path Accumulated Metric: 140

103128 [Prefix-SID, 3.3.3.3]

105128 [Prefix-SID, 5.5.5.5]

108128 [Prefix-SID, 8.8.8.8]

Attributes:

Binding SID: 119076

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
 | Output from R2 is similar & Output for SR Policy to 4.4.4.4 is unaffected, omitted for brevity. |

On CE1, traceroute to CE3 to display the new path taken.

RP/0/0/CPU0:CE1# **traceroute vrf CUSTOMER-B 150.23.7.7 probe 1**

Type escape sequence to abort.

Tracing the route to 150.23.7.7

1 r2 (172.2.21.100) 9 msec

2 pce1 (172.2.11.1) [MPLS: Labels 103128/105128/108128/119007 Exp 0] 239 msec

3 r3 (172.3.11.0) [MPLS: Labels 105128/108128/119007 Exp 0] 29 msec

4 r5 (172.3.5.1) [MPLS: Labels 108128/119007 Exp 0] 39 msec

5 pce3 (172.5.13.1) [MPLS: Labels 108128/119007 Exp 0] 19 msec

6 r8 (172.8.13.0) [MPLS: Label 119007 Exp 0] 9 msec

7 ce3 (172.8.23.101) 9 msec

| ! |
 |
 |
| --- | --- | --- |
|

NOTE

 |
 | Your lab output may be different if the ECMP hashed to R2 instead, output using R2 is omitted for brevity. |

## Task 10: Restore original Latency value

On PCE2 restore the delay to R5 as 10ms and to R6 as 13ms.

**performance-measurement**

**interface GigabitEthernet0/0/0/2**

**delay-measurement**

**advertise-delay 10**

**!**

**!**

**interface GigabitEthernet0/0/0/3**

**delay-measurement**

**advertise-delay 13**

**!**

**!**

**!**

# Scenario 10 - Configure Flex Algo for Dual Plane

One use case of Flex-Algo involve dual plane disjoint paths. Below topology consists of two planes:

User Plane 1 – Flex Algo 129 – PCE1, R3, PCE2, R5, PCE3

User Plane 2 – Flex Algo 130 – PCE1, R4, PCE2, R6, PCE3

Both algorithms have the same definition: optimize IGP metric, no constraints. By using Flex Algorithm, operators can also join completely disjoint path. However, in the below topology devices PCE1, PCE2, PCE3 & Services nodes R1, R2, R7, R8 are participating in both user planes 1 & 2.

![](RackMultipart20230222-1-cgwn2a_html_446d077a454a7cca.png)

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

6 ![](RackMultipart20230222-1-cgwn2a_html_7e7316a3bd6bf21f.png)| Page
