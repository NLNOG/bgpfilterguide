---
layout: page
title: Bogon Prefixes
permalink: /guides/bogon_prefixes/
---

* TOC
{:toc}

# Filter Bogon prefixes

## Purpose

These prefixes are not globally unique prefixes. IETF didn't intend for
these to be routed on the public Internet.

# Configuration Examples IPv4

## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L97-L109)

```
deny from any prefix 0.0.0.0/8 prefixlen >= 8		# 'this' network [RFC1122]
deny from any prefix 10.0.0.0/8 prefixlen >= 8		# private space [RFC1918]
deny from any prefix 100.64.0.0/10 prefixlen >= 10	# CGN Shared [RFC6598]
deny from any prefix 127.0.0.0/8 prefixlen >= 8 	# localhost [RFC1122]
deny from any prefix 169.254.0.0/16 prefixlen >= 16	# link local [RFC3927]
deny from any prefix 172.16.0.0/12 prefixlen >= 12	# private space [RFC1918]
deny from any prefix 192.0.2.0/24 prefixlen >= 24	# TEST-NET-1 [RFC5737]
deny from any prefix 192.168.0.0/16 prefixlen >= 16	# private space [RFC1918]
deny from any prefix 198.18.0.0/15 prefixlen >= 15	# benchmarking [RFC2544]
deny from any prefix 198.51.100.0/24 prefixlen >= 24	# TEST-NET-2 [RFC5737]
deny from any prefix 203.0.113.0/24 prefixlen >= 24	# TEST-NET-3 [RFC5737]
deny from any prefix 224.0.0.0/4 prefixlen >= 4 	# multicast
deny from any prefix 240.0.0.0/4 prefixlen >= 4 # reserved
```
## Junos
```
policy-options {
    prefix-list BOGONS_v4 {
        0.0.0.0/8;
        10.0.0.0/8;
        100.64.0.0/10;
        127.0.0.0/8;
        169.254.0.0/16;
        172.16.0.0/12;
        192.0.2.0/24;
        192.168.0.0/16;
        198.18.0.0/15;
        198.51.100.0/24;
        203.0.113.0/24;
        224.0.0.0/4;
        240.0.0.0/4;
    }
    policy-statement BGP_FILTER_IN {
        term IPv4 {
            from {
                prefix-list BOGONS_v4;
            }
            then reject;
        }
    }
}
```

## Bird
```
prefix set bogon; {
        bogon = [ 0.0.0.0/8+, 10.0.0.0/8+, 100.64.0.0/10+, 127.0.0.0/8+, 169.254.0.0/16+,  172.16.0.0/12+, 192.0.2.0/24+, 192.168.0.0/16+, 198.18.0.0/15+, 198.51.100.0/24+, 203.0.113.0/24+, 224.0.0.0/4+, 240.0.0.0/4+ ];
        if ( net ~ bogon ) then  {
          return false;
        }
        return true;
}
```
# Configuration Examples IPv6

## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L111-L121)

```
deny from any prefix ::/8 prefixlen >= 8
deny from any prefix 0100::/64 prefixlen >= 64          # Discard-Only [RFC6666]
deny from any prefix 2001:2::/48 prefixlen >= 48        # BMWG [RFC5180]
deny from any prefix 2001:10::/28 prefixlen >= 28       # ORCHID [RFC4843]
deny from any prefix 2001:db8::/32 prefixlen >= 32      # docu range [RFC3849]
deny from any prefix 3ffe::/16 prefixlen >= 16          # old 6bone
deny from any prefix fc00::/7 prefixlen >= 7            # unique local unicast
deny from any prefix fe80::/10 prefixlen >= 10          # link local unicast
deny from any prefix fec0::/10 prefixlen >= 10          # old site local unicast
deny from any prefix ff00::/8 prefixlen >= 8            # multicast
```

## Juniper and Cisco

Gert Doering's [ipv6-filters](https://www.space.net/~gert/RIPE/ipv6-filters.html)

## YAML from Coloclue

Coloclue's network management system [kees](https://github.com/coloclue/kees) considers these the IPv6 Bogons: [yaml file](https://github.com/coloclue/kees/blob/master/vars/generic.yml#L70-L156)
