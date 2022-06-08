## Scenario 5 - Inter-Domain Network Slicing for TE-Metric

### Task 1: Configure color for TE-metric

#### Apply the following configuration in R7 and R8:

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

### Task 3: Configure SRTE policy for TE-metric

#### On R1 & R2 apply the following configuration:

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
```

### Task 5: Change TE metric of a link

#### On R1 and R2 issue the following command:

```
segment-routing
 traffic-eng
  interface GigabitEthernet0/0/0/3
   metric 5000
!
```

### Task 7: Restore original TE metric

#### On R1 and R2 issue the following command:

```
segment-routing
 traffic-eng
  interface GigabitEthernet0/0/0/3
   metric 1000
!
```
