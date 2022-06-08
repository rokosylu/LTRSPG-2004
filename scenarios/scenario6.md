## Scenario 6 - Inter-Domain Network Slicing for Regular Traffic with Anycast SID

### Task 1: Configure color for Anycast SID

#### Apply the following configuration in R7 and R8:

```
extcommunity-set opaque COLOR-3235
 3235
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
  ##### Everything Else #####
 else
pass
 endif
end-policy
```

### Task 3: Configure Anycast-SID at ABRs

#### Apply the following configuration in R5 and R6:

```
interface Loopback56
 ipv4 address 56.56.56.56 255.255.255.255
!
router isis ACCESS-2
 interface Loopback56
  address-family ipv4 unicast
  tag 100
  prefix-sid index 56 n-flag-clear
!
router isis AGG-CORE
 interface Loopback56
  address-family ipv4 unicast
   tag 100
   prefix-sid index 56 n-flag-clear
!
```

### Task 4: Configure SR-TE Policy using anycast SID

#### On R1 & R2 apply the following configuration:

```
segment-routing
 traffic-eng
  policy ANYCAST-IGP_COLOR-3235_R7
   color 3235 end-point ipv4 7.7.7.7
   candidate-paths
    preference 100
     dynamic
      pcep
      !
      metric
       type igp
      !
      anycast-sid-inclusion
      !
   policy ANYCAST-IGP_COLOR-3235_R8
   color 3235 end-point ipv4 8.8.8.8
    candidate-paths
     preference 100
      dynamic
       pcep
       !
      metric
       type igp
      !
      anycast-sid-inclusion
      !
```




