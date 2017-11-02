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

The IETF document that defines GRACEFUL_SHUTDOWN: [RFC-to-Be draft-ietf-grow-bgp-gshut](https://tools.ietf.org/html/draft-ietf-grow-bgp-gshut-12).

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

## Junos

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

# List of networks known to accept & honor GRACEFUL_SHUTDOWN

* NTT Communications / AS 2914
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
