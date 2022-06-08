## Scenario 8 – Intra and Inter-Domain Network Slicing for On-Demand Next Hop (ODN)

### Task 1: Configure color for ODN

#### Apply the following configuration in R4:

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

#### Apply the following configuration in R7 and R8:

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

### Task 3: Configure SRTE policy to use ODN vs defined end point.

#### On R1 and R2 issue the following command:

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
```
