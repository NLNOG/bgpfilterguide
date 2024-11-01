---
layout: page
title: BGP Graceful Shutdown
permalink: /guides/graceful_shutdown/
---

* TOC
{:toc}

# Graceful BGP session shutdown

## Purpose

A new well-known BGP community GRACEFUL_SHUTDOWN (65535:0) to signal the
graceful shutdown of paths has been introduced by the IETF. The purpose of this
community is to reduce the amount of traffic lost when BGP peering sessions are
about to be shut down deliberately, e.g. for planned maintenance.

## Further reading

**Recommended viewing**: 15 minute presentation available as [youtube video](https://www.youtube.com/watch?v=HGGRsJ-gjI4) on GRACEFUL_SHUTDOWN by
Job Snijders, recorded at the [NLNOG day 2017](https://nlnog.net/nlnog-day-2017/).

The IETF document that defines GRACEFUL_SHUTDOWN: [RFC 8326](https://tools.ietf.org/html/rfc8326).

The requirements document which led to the development of GRACEFUL_SHUTDOWN: [RFC 6198](https://tools.ietf.org/html/rfc6198).

European Peering Forum 12 (2017, Lisbon) presentation by Job Snijders on Graceful Shutdown: [pdf](https://www.peering-forum.eu/system/documents/173/original/Job_Snijders_BGP_graceful_shutdown.pdf).

# Configuration Examples

## IOS XR

```
community-set comm-graceful-shutdown
  65535:0
end-set
!
route-policy AS64497-ebgp-inbound
  ! normally this policy would contain much more
  if community matches-any comm-graceful-shutdown then
    set local-preference 0
  endif
end-policy
!
router bgp 64496
 neighbor 2001:db8:1:2::1
  remote-as 64497
  address-family ipv6 unicast
   send-community-ebgp
   route-policy AS64497-ebgp-inbound in
   ! outbound policy & other stuff omitted in this example
  !
 !
!
```

## OpenBSD OpenBGPD

```
AS 64496
router-id 192.0.2.1
neighbor 2001:db8:1:2::1 {
	remote-as 64497
}
# normally this policy would contain much more
match from any community GRACEFUL_SHUTDOWN set { localpref 0 }
```

## FRR (vtysh)

```
router bgp <own_ASN>
  bgp graceful-shutdown
```
This will enable sending of the graceful shutdown community on all prefixes to all peers.

## VyOS
Older Versions:
```
set policy community-list GSHUT description 'RFC 8326 -- Graceful Shutdown'
set policy community-list GSHUT rule 10 action 'permit'
set policy community-list GSHUT rule 10 regex '65535:0'

set policy route-map TRANSIT-IN rule 10 action 'permit'
set policy route-map TRANSIT-IN rule 10 match community community-list 'GSHUT'
set policy route-map TRANSIT-IN rule 10 set local-preference '0'
```

Modern Versions:
```
set protocols bgp parameters graceful-shutdown
```

## Junos

Junos supports graceful shutdown by default [as of version 19.1](https://www.juniper.net/documentation/en_US/junos/topics/reference/configuration-statement/graceful-shutdown-edit-protocols-bgp.html). Local preference can be set to any value. For previous versions, or if one does not want to use the provided feature, it still possible to configure it with a policy.

Put `allow-graceful-shutdown` in every import EBGP policy chain. Ensure this is placed in a location after you've decided to accept a route, and the `local-preference 0` is not overwriten later on in the chain.

```
community graceful_shutdown members 65535:0;

policy-statement allow-graceful-shutdown {
  term 1 {
    from {
      protocol bgp;
      community graceful_shutdown;
    }
    then {
      local-preference 0;
      next policy;
    }
  }
}
```

## BIRD
```
function honor_graceful_shutdown() {
  if (65535, 0) ~ bgp_community then {
    bgp_local_pref = 0;
  }
}

filter AS64497_ebgp_inbound
{
  # normally this policy would contain much more
  honor_graceful_shutdown();
}

protocol bgp peer_64497_1 {
  neighbor 2001:db8:1:2::1 as 64497;
  local as 64496;
  import keep filtered;
  import filter AS64497_ebgp_inbound;
}
```

## Arista
Arista supports the well-known BGP community GRACEFUL_SHUTDOWN by default (even when the switch is not in maintenance mode), and sets the Local Preference to 0 for routes received with the GRACEFUL_SHUTDOWN community.

## Brocade / Cisco IOS / Quagga / FRR

```
!
ip community-list standard gshut 65535:0
!
route-map ebgp-in permit 10
  match community gshut
  set local-preference 0
  continue
!
```

## Nokia SR OS
```
#
# Classic CLI
#
#--------------------------------------------------
echo "Policy Configuration"
#--------------------------------------------------
        policy-options
            begin
            community "gshut"
                members "65535:0"
            exit
            policy-statement "BGP_FILTER_IN"
                entry 60
                    from
                        community "gshut"
                    exit
                    action accept
                        local-preference 0
                    exit
                exit
            exit
            commit
        exit

#
# MD-CLI
#
[gl:configure policy-options]
community "gshut" {
    member "65535:0" { }
}
policy-statement "BGP_FILTER_IN" {
    entry 60 {
        from {
            community {
                name "gshut"
            }
        }
        action {
            action-type accept
            local-preference 0
        }
    }
}
```
## FortiOS
```
config router community-list
    edit "gshut"
        set type standard
        config rule
            edit 1
                set action permit
                set match "65535:0"
            next
        end
    next
end

config router route-map
    edit "BGP_FILTER_IN"
        config rule
            edit 1
                set action permit
                set match-community "gshut"
		set set-local-preference 0
            next
        end
    next
end
```


## MikroTik 
### RouterOS v7
RouterOS 7 added support for several Well-known communities including graceful-shutdown. 
```
/routing/filter/rule 
add chain="GENERIC_PREFIX_LIST" rule="if (bgp-communities includes graceful-shutdown) { set bgp-local-pref 0; }"
```

## Huawei VRP
```
ip community-filter basic GRACEFUL-SHUTDOWN index 10 permit 65535:0

route-policy TRANSIT-IN permit node 350
 if-match community-filter GRACEFUL-SHUTDOWN
 apply local-preference 0
 ```

# List of networks known to accept & honor GRACEFUL_SHUTDOWN

* NTT Ltd (Global IP Network) / AS 2914
* GTT Communications / AS 3257
* Github.com / AS 36459
* Nordunet / AS 2603
* Coloclue / AS 8283
* Amsio / AS 8315
* BIT / AS 12859
* Telia / AS 3301 + AS 1299
* Tele2 / AS 1257
* SVT / AS 201641
* Netnod / AS 8674
* Bahnhof / AS 8473
* DGC Systems / AS 21195
* ComHem / AS 39651
* Digital Telecommunication Services S.r.l. / AS 49605
* Job Snijders / AS 15562
* Thomas King / AS 31451
* KPN Eurorings (International) / AS 286
* The Wikimedia Foundation / AS 14907
* DigitalOcean / AS 14061
* Atom86 / AS 8455
* Sky Italia / AS 210278
* Fiber Telecom S.p.A. / AS 41327
* PCCW / AS 3491
* TDC / AS 3292
* HOPUS / AS 44530
* Fastly / AS 54113
* Interconnect Services BV / AS 9150
* Fusix Networks / AS 57866
* Dyjix / AS 212815
* Cogent / AS 174
* Booking.com / AS 43996 + AS 202196
