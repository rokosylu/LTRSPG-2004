## Scenario 9 - Configure Flex Algo & ODN for low latency

### Task 1: Configure Flex Algo 128 for low latency and a prefix SID on the ABR Nodes

#### Apply the following configuration in R3:

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
```

#### Apply the following configuration in R4:

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
```

#### Apply the following configuration in R5:

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
```

#### Apply the following configuration in R6:

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
```

#### Apply the following configuration in R1:

```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 101128
!
```

#### Apply the following configuration in R2:

```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 102128
!
```

#### Apply the following configuration in PCE1:

```
router isis ACCESS-1
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 111128
!
```

#### Apply the following configuration in PCE2:

```
router isis AGG-CORE
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 112128
!
```

#### Apply the following configuration in PCE3:

```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 113128
!
```

#### Apply the following configuration in R7:

```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 107128
!
```

#### Apply the following configuration in R8:

```
router isis ACCESS-2
 flex-algo 128
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 128 absolute 108128
!
```

### Task 4: RPL to Color CE prefixes of VRF CUSTOMER-B on R4, R7 & R8

#### Apply the following configuration in R4:

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
```

#### Apply the following configuration in R7:

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
```

#### Apply the following configuration in R8:

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
```

### Task 6: ODN and flex Algo 128 policy on R1 & R2

### Apply the following configuration in R1 & R2:

```
segment-routing
 traffic-eng
  on-demand color 4128
   dynamic
    pcep
    !
    sid-algorithm 128
!
```

### Task 8: Change Latency value of a link

#### On PCE2 configure the delay to R5 and R6 as 150ms.

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
```

### Task 10: Restore original Latency value

#### On PCE2 restore the delay to R5 as 10ms and to R6 as 13ms.

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
```


