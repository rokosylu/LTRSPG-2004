# Scenario 10 - Configure Flex Algo for Dual Plane

Task 1: Configure Flex Algo for User plane-1

Apply the following configuration in R3:

'''
router isis ACCESS-1
 flex-algo 129
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 103129
  !
 !
!
router isis AGG-CORE
 flex-algo 129
  advertise-definition
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 103129
'''


! Apply the following configuration in R5:

router isis ACCESS-1
 flex-algo 129
  advertise-definition
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 105129
  !
 !
!
router isis AGG-CORE
 flex-algo 129
  advertise-definition
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 105129



! Task 2: Configure Flex Algo for User plane-2

! Apply the following configuration in R4:

router isis ACCESS-1
 flex-algo 130
  advertise-definition
  !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 130 absolute 104130
  !
 !
!
router isis AGG-CORE
 flex-algo 130
  advertise-definition
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 130 absolute 104130
!



! Apply the following configuration in R6:

router isis ACCESS-2
 flex-algo 130
  advertise-definition
  !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 130 absolute 106130
  !
 !
!
router isis AGG-CORE
 flex-algo 130
  advertise-definition
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 130 absolute 106130
!
!
!

! Task 3: Configure Flex Algo for User plane-1 & 2

! Apply the following configuration in R1:

router isis ACCESS-1
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 101129
   prefix-sid algorithm 130 absolute 101130
!

! Apply the following configuration in R2:

router isis ACCESS-1
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 102129
   prefix-sid algorithm 130 absolute 102130
  !
 !


! Apply the following configuration in PCE1:

router isis ACCESS-1
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 21029
   prefix-sid algorithm 130 absolute 21030
  !
 !

! Apply the following configuration in PCE2:

router isis AGG-CORE
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 112129
   prefix-sid algorithm 130 absolute 112130
  !

! Apply the following configuration in PCE3:

router isis ACCESS-2
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 113129
   prefix-sid algorithm 130 absolute 113130
  !

! Apply the following configuration in R7:

router isis ACCESS-2
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 107129
   prefix-sid algorithm 130 absolute 107130
  !
 !


Apply the following configuration in R8:

router isis ACCESS-2
 flex-algo 129
 flex-algo 130
!
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid algorithm 129 absolute 108129
   prefix-sid algorithm 130 absolute 108130
  !
 !




! Task 6: Configure RPL to Color CE prefixes of VRF CUSTOMER-B

! Apply the following configuration in R7:

extcommunity-set opaque COLOR-4129
  4129
end-set
!
extcommunity-set opaque COLOR-4130
  4130
end-set
!
route-policy CUST-B_SET_COLOR_IN
  ##### ODN Flex Algo 128 - Latency #####
  if destination in (150.23.7.7) then
    set extcommunity color COLOR-4128
  ##### ODN Flex Algo 129 - User Plane1 #####
  elseif destination in (150.23.1.1) then
    set extcommunity color COLOR-4129
  ##### ODN Flex Algo 130 - User Plane2 #####
  elseif destination in (150.23.2.2) then
    set extcommunity color COLOR-4130
  ##### Everything Else #####
  else
    pass
  endif
end-policy
!


! Apply the following configuration in R8:

extcommunity-set opaque COLOR-4129
  4129
end-set
!
extcommunity-set opaque COLOR-4130
  4130
end-set
!
route-policy CUST-B_SET_COLOR_IN
  ##### ODN Flex Algo 128 - Latency #####
  if destination in (150.23.7.7) then
    set extcommunity color COLOR-4128
  ##### ODN Flex Algo 129 - User Plane1 #####
  elseif destination in (150.23.1.1) then
    set extcommunity color COLOR-4129
  ##### ODN Flex Algo 130 - User Plane2 #####
  elseif destination in (150.23.2.2) then
    set extcommunity color COLOR-4130
 
  ##### Everything Else #####
  else
    pass
  endif
end-policy



! Task 8: ODN and flex Algo 129 & 130 policy on R1 & R2

! Apply the following configuration in R1 & R2:

segment-routing
 traffic-eng
  on-demand color 4129
   dynamic
    pcep
    !
    sid-algorithm 129

  on-demand color 4130
   dynamic
    pcep
    !
    sid-algorithm 130
   !
  !
 !





