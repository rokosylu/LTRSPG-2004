## Scenario 3 - Inter-domain SRTE using Explicit-path

### Task 1: Configure color for Explicit-path

#### Apply the following configuration in R4:

```
extcommunity-set opaque COLOR-3222
  3222
end-set
```

#### Apply the following configuration in R7 and R8:

```
extcommunity-set opaque COLOR-3232
 3232
end-set
```

#### Apply the following configuration in R4:

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

router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.4.22.1
   address-family ipv4 unicast
   route-policy CUST-A_SET_COLOR_IN in
```
   
#### Apply the following configuration in R7 and R8:

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

#### Apply the following configuration in R7:

```
router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.7.23.1
   address-family ipv4 unicast
   route-policy CUST-A_SET_COLOR_IN in

#### Apply the following configuration in R8:
router bgp 65001
 vrf CUSTOMER-A
  neighbor 172.8.23.1
   address-family ipv4 unicast
   route-policy CUST-A_SET_COLOR_IN in
```

### Task 3: Configure SRTE policy using Explicit-path

#### On R1 & R2 apply the following configuration:

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
      explicit segmnt-list PCE1-R3-R4
!

  policy EXP_COLOR-3232_R7
   color 3232 end-point ipv4 7.7.7.7
    candidate-paths
     preference 100
      explicit segment-list R3-R4-R6-R5-R7
!

  policy EXP_COLOR-3232_R8
   color 3232 end-point ipv4 8.8.8.8
    candidate-paths
     preference 100
      explicit segment-list R4-R3-R5-R6-R8
!
```

