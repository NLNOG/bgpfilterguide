---
layout: page
title: BLACKHOLE BGP Community
permalink: /guides/blackhole_bgp_community/
---

* TOC
  {:toc}

# Trigger blackholing via BGP communities

## Purpose

To clearly announce single IP addresses or very specific towards upstream providers and peers, such announcements can be configured to carry some communities. This can be of use e.g. during a DDoS to null-route a single IP address while keeping the rest of the network reachable.

Peers can then evaluate this information and proceed to doing the needed things - e.g. null-route traffic to this IP address or prefix to actually blackhole this target.

What BGP community is treated as trigger to blackhole traffic is depending on the receiving AS'es routing policy. But with [RFC7999](https://www.rfc-editor.org/rfc/rfc7999) a standard evolved and many networks treat the general BLACKHOLE community `65535:666` as such.

Announcements for blackholing are often a `/32` (IPv4) or a `/128` (IPv6) that would normally get filtered out because they are too specific. But in case the blackhole flags are set, these routes should be accepted by the peers independent of the prefix length.

# Configuration Examples

## BIRD v2.x
### Define which IP address(es) to blackhole
```
# BLACKHOLING the following routes:
protocol static blackhole_v4 {
	ipv4;

	route x.x.x.99/32 via y.y.y.y;
}
```

### Define a filter that can be applied multiple times
```
# filter for our own networks to be exported to peers + upstreams
filter own_nets {
	# BLACKHOLING of IPs
	if (proto = "blackhole_v4") then {

		print "BLACKHOLING: ", net;

		# Add well-known RFC7999 BLACKHOLE community
		bgp_community.add((65535, 666));
		
		# Add further communities depending on your peers
		# configuration recommendations. E.g. for AS3303 (Swisscom) it's "3303:888"
		
		# Swisscom Blackholing
		bgp_community.add((3303, 888));
		
		accept;
	}

	# our networks that shall pass this filter (and get announced)
	if net ~ [ x.x.x.x/24 ] then { accept; }

	reject;
}
```

### Use this filter in BGP peer config
```
protocol bgp upstream_a_v4 {
  (...)
  ipv6 {
    (...)
    export filter own_nets;
  };
}
```
