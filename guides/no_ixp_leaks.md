---
layout: page
title: Filtering IXP Peering LANs
permalink: /guides/no_ixp_leaks/
---

* TOC
{:toc}

# Filter IXP Peering LANs

## Purpose

IXP Peering LAN prefixes may be advertised by IXPs from their services ASN. Some IXPs prefer not to advertise their Peering LANs at all.
If you are connected to an IXP you should never accept the IXP's prefix from any other neighbor than the IXP itself.

Accepting Peering LAN prefixes can be especially dangerous when an IXP increases their Peering LAN subnet size.
If the local IXP-connected router has the increased subnet size configured and a route with the older, more specific subnet is received via BGP this can lead to instability.

When the IXP has created ROAs and [has not configured](https://datatracker.ietf.org/doc/html/rfc9319) the maxLength attribute, [rejecting RPKI invalid routes](/guides/reject_invalids) offers sufficient protection.
The examples below are meant for scenario's where there is no ROA, or in case you have not implemented RPKI ROV.

# Configuration Examples

## OpenBGPD

```
deny quick from any prefix 80.249.208.0/21 or-longer
deny quick from any prefix 193.239.116.0/22 or-longer
```

## BIRD

```
function net_ixp() {
    case net.type {
        NET_IP4:
            return net ~ [
                80.249.208.0/21+,  # AMS-IX
                193.239.116.0/22+  # NL-IX
            ];
        NET_IP6:
            return net ~ [
                2001:7f8:1::/64+,  # AMS-IX
                2001:7f8:13::/64+  # NL-IX
            ];
    }
}

template bgp peer_v4 {
    ipv4 {
        import keep filtered;
        import filter {
            if net_ixp() then reject;
        }
    }
}
template bgp peer_v6 {
    ipv4 {
        import keep filtered;
        import filter {
            if net_ixp() then reject;
        }
    }
}
```

## Junos

```
policy-options {
    prefix-list ixpnets {
        80.249.208.0/21;
        193.239.116.0/22;
    }
    prefix-list v6-ixpnets {
        2001:7f8:1::/64;
        2001:7f8:13::/64;
    }
    policy-statement bgp-import-policy {
        term no-ixp-leaks {
            from {
                prefix-list-filter ixpnets orlonger;
            }                               
            then reject;                    
        }                                   
        term no-v6-ixp-leaks {
            from {
                prefix-list-filter v6-ixpnets orlonger;
            }                               
            then reject;
        }
    }                    
}
```

## FRR (vtysh)

```
ip prefix-list IXPNET4 permit 80.249.208.0/21 ge 21 le 32
ip prefix-list IXPNET4 permit 193.239.116.0/22 ge 22 le 32
!
ipv6 prefix-list IXPNET6 permit 2001:7f8:1::/64 ge 64 le 128
ipv6 prefix-list IXPNET6 permit 2001:7f8:13::/64 ge 64 le 128
!
route-map BGP_FILTER_IN deny 21
    match ip address prefix-list IXPNET4
!
route-map BGP_FILTER_IN deny 22
    match ipv6 address prefix-list IXPNET6
!
```

## VyOS
```
set policy prefix-list IXP-LANS-V4 rule 10 action 'permit'
set policy prefix-list IXP-LANS-V4 rule 10 ge '24'
set policy prefix-list IXP-LANS-V4 rule 10 le '32'
set policy prefix-list IXP-LANS-V4 rule 10 prefix '192.33.255.0/24'
set policy prefix-list6 IXP-LANS-V6 rule 10 action 'permit'
set policy prefix-list6 IXP-LANS-V6 rule 10 ge '48'
set policy prefix-list6 IXP-LANS-V6 rule 10 le '128'
set policy prefix-list6 IXP-LANS-V6 rule 10 prefix '2001:504:30::/48'

set policy route-map INTERNET-IN rule 10 action 'deny'
set policy route-map INTERNET-IN rule 10 match ip address prefix-list 'IXP-LANS-V4'
set policy route-map INTERNET-IN rule 20 action 'deny'
set policy route-map INTERNET-IN rule 20 match ipv6 address prefix-list 'IXP-LANS-V6'
```

## Arista

```
ip prefix-list IXPNET4 seq 101 permit 80.249.208.0/21 le 32
ip prefix-list IXPNET4 seq 102 permit 193.239.116.0/22 le 32
!
ipv6 prefix-list IXPNET6
    seq 101 permit 2001:7f8:1::/64 le 128
    seq 102 permit 2001:7f8:13::/64 le 128
!
route-map BGP_FILTER_IN deny 21
    match ip address prefix-list IXPNET4
!
route-map BGP_FILTER_IN deny 22
    match ipv6 address prefix-list IXPNET6
!
```
