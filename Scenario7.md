## Scenario 7 - Inter-Domain Network Slicing with link affinities

### Task 1: Configure color

#### Apply the following configuration in R7 and R8:

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

### Task 3: Configure color to avoid using links with marked affinity (red link)

#### On R1 and R2 issue the following command:

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
```

### Task 4: Configure SRTE policy to avoid red links

#### On R1 and R2 issue the following command:

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
root
!
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
```
