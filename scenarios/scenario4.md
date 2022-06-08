## Scenario 4 - Inter-Domain Network Slicing for Low Latency 

### Task 1: Configure color Low Latency Traffic

#### Apply the following configuration in R7 and R8:

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

### Task 3: Configure SRTE policy Low Latency Traffic 


#### On R1 & R2 apply the following configuration:

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

  policy DYN-Latency_COLOR-3233_R8
   color 3233 end-point ipv4 8.8.8.8
    candidate-paths
     preference 100
      dynamic
       pcep
       !
       metric
        type latency
```
        
### Task 5: Change the Latency value of a link

#### On PCE2 increase the delay to R5 and R6 from 10ms to 150ms.

```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 150
!

interface GigabitEthernet0/0/0/3
 delay-measurement
   advertise-delay 150
!
```

### Task 7: Restore the original Latency value

#### On PCE2 restore the delay to R5 and R6 from 150ms to 10ms & 13ms respectively.

```
performance-measurement
 interface GigabitEthernet0/0/0/2
  delay-measurement
   advertise-delay 10
!

interface GigabitEthernet0/0/0/3
 delay-measurement
   advertise-delay 13
!
```
