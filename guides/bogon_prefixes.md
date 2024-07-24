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

## BIRD
```
define BOGON_PREFIXES = [
  0.0.0.0/8+,         # RFC 1122 'this' network
  10.0.0.0/8+,        # RFC 1918 private space
  100.64.0.0/10+,     # RFC 6598 Carrier grade nat space
  127.0.0.0/8+,       # RFC 1122 localhost
  169.254.0.0/16+,    # RFC 3927 link local
  172.16.0.0/12+,     # RFC 1918 private space
  192.0.2.0/24+,      # RFC 5737 TEST-NET-1
  192.88.99.0/24+,    # RFC 7526 6to4 anycast relay
  192.168.0.0/16+,    # RFC 1918 private space
  198.18.0.0/15+,     # RFC 2544 benchmarking
  198.51.100.0/24+,   # RFC 5737 TEST-NET-2
  203.0.113.0/24+,    # RFC 5737 TEST-NET-3
  224.0.0.0/4+,       # multicast
  240.0.0.0/4+ ];     # reserved

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
  bogon_prefixes = BOGON_PREFIXES;

  if (net ~ bogon_prefixes) then {
    print "Reject: Bogon prefix: ", net, " ", bgp_path;
    reject;
  }
}

...

filter transit_in {
  reject_invalids();
  reject_bogon_asns();
  reject_bogon_prefixes();
  reject_long_aspaths();
  reject_small_prefixes();
  reject_default_route();

...

  honor_graceful_shutdown();
  accept;
}

filter ixp_in {
  reject_invalids();
  reject_bogon_asns();
  reject_bogon_prefixes();
  reject_long_aspaths();
  reject_transit_paths();
  reject_small_prefixes();
  reject_default_route();

...

  honor_graceful_shutdown();
  accept;
}

```
## FortiOS
```
config router prefix-list
    edit "IPv4_BOGONS"
        config rule
            edit 1
                set prefix 0.0.0.0 255.0.0.0
                set ge 9
                unset le
            next
            edit 3
                set prefix 100.64.0.0 255.192.0.0
                set ge 11
                unset le
            next
            edit 2
                set prefix 10.0.0.0 255.0.0.0
                set ge 9
                unset le
            next
            edit 4
                set prefix 127.0.0.0 255.0.0.0
                set ge 9
                unset le
            next
            edit 5
                set prefix 169.254.0.0 255.255.0.0
                set ge 17
                unset le
            next
            edit 6
                set prefix 172.16.0.0 255.240.0.0
                set ge 13
                unset le
            next
            edit 7
                set prefix 192.0.2.0 255.255.255.0
                unset ge
                unset le
            next
            edit 8
                set prefix 192.88.99.0 255.255.255.0
                unset ge
                unset le
            next
            edit 9
                set prefix 192.168.0.0 255.255.0.0
                set ge 17
                unset le
            next
            edit 10
                set prefix 198.18.0.0 255.254.0.0
                set ge 16
                unset le
            next
            edit 11
                set prefix 198.51.100.0 255.255.255.0
                unset ge
                unset le
            next
            edit 12
                set prefix 203.0.113.0 255.255.255.0
                unset ge
                unset le
            next
            edit 13
                set prefix 224.0.0.0 240.0.0.0
                set ge 5
                unset le
            next
            edit 14
                set prefix 240.0.0.0 240.0.0.0
                set ge 5
                unset le
            next
        end
    next
end

config router route-map
    edit "BGP_FILTER_IN"
        config rule
            edit 1
                set action deny
                set match-ip-address "IPv4_BOGONS"
            next
        end
    next
end
```
## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L97-L109)

```
deny from any prefix 0.0.0.0/8 prefixlen >= 8           # 'this' network [RFC1122]
deny from any prefix 10.0.0.0/8 prefixlen >= 8          # private space [RFC1918]
deny from any prefix 100.64.0.0/10 prefixlen >= 10      # CGN Shared [RFC6598]
deny from any prefix 127.0.0.0/8 prefixlen >= 8         # localhost [RFC1122]
deny from any prefix 169.254.0.0/16 prefixlen >= 16     # link local [RFC3927]
deny from any prefix 172.16.0.0/12 prefixlen >= 12      # private space [RFC1918]
deny from any prefix 192.0.2.0/24 prefixlen >= 24       # TEST-NET-1 [RFC5737]
deny from any prefix 192.88.99.0/24 prefixlen >= 24     # 6to4 anycast relay [RFC7526]
deny from any prefix 192.168.0.0/16 prefixlen >= 16     # private space [RFC1918]
deny from any prefix 198.18.0.0/15 prefixlen >= 15      # benchmarking [RFC2544]
deny from any prefix 198.51.100.0/24 prefixlen >= 24    # TEST-NET-2 [RFC5737]
deny from any prefix 203.0.113.0/24 prefixlen >= 24     # TEST-NET-3 [RFC5737]
deny from any prefix 224.0.0.0/4 prefixlen >= 4         # multicast
deny from any prefix 240.0.0.0/4 prefixlen >= 4         # reserved for future use
```

## FRR (vtysh)
```
ip prefix-list BOGONS_v4 deny 0.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 10.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 100.64.0.0/10 le 32
ip prefix-list BOGONS_v4 deny 127.0.0.0/8 le 32
ip prefix-list BOGONS_v4 deny 169.254.0.0/16 le 32
ip prefix-list BOGONS_v4 deny 172.16.0.0/12 le 32
ip prefix-list BOGONS_v4 deny 192.0.2.0/24 le 32
ip prefix-list BOGONS_v4 deny 192.88.99.0/24 le 32
ip prefix-list BOGONS_v4 deny 192.168.0.0/16 le 32
ip prefix-list BOGONS_v4 deny 198.18.0.0/15 le 32
ip prefix-list BOGONS_v4 deny 198.51.100.0/24 le 32
ip prefix-list BOGONS_v4 deny 203.0.113.0/24 le 32
ip prefix-list BOGONS_v4 deny 224.0.0.0/4 le 32
ip prefix-list BOGONS_v4 deny 240.0.0.0/4 le 32
```

## Mikrotik

### RouterOS v6
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=0.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1122 'this' network"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=10.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=100.64.0.0/10 prefix-length=10-32 protocol=bgp action=discard comment="RFC 6598 Carrier grade nat space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=127.0.0.0/8 prefix-length=8-32 protocol=bgp action=discard comment="RFC 1122 localhost"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=169.254.0.0/16 prefix-length=16-32 protocol=bgp action=discard comment="RFC 3927 link local"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=172.16.0.0/12 prefix-length=12-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.0.2.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-1"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.88.99.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 7526 6to4 anycast relay"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=192.168.0.0/16 prefix-length=16-32 protocol=bgp action=discard comment="RFC 1918 private space"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=198.18.0.0/15 prefix-length=15-32 protocol=bgp action=discard comment="RFC 2544 benchmarking"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=198.51.100.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-2"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=203.0.113.0/24 prefix-length=24-32 protocol=bgp action=discard comment="RFC 5737 TEST-NET-3"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=224.0.0.0/4 prefix-length=4-32 protocol=bgp action=discard comment="multicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ip prefix=240.0.0.0/4 prefix-length=4-32 protocol=bgp action=discard comment="multicast"
```

### RouterOS v7
```
/routing/filter/rule
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==0.0.0.0/8 && dst-len >= 8 ){ reject; }" comment="RFC 1122 'this' network"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==10.0.0.0/8 && dst-len >= 8){ reject; }" comment="RFC 1918 private space"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==100.64.0.0/10 && dst-len >= 10){ reject; }" comment="RFC 6598 Carrier grade nat space"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==127.0.0.0/8 && dst-len >= 8){ rejecet; }" comment="RFC 1122 localhost"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==169.254.0.0/16 && dst-len >= 16){ reject; }" comment="RFC 3927 link local"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==172.16.0.0/12 && dst-len >= 12){ reject; }" comment="RFC 1918 private space"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==192.0.2.0/24 && dst-len >= 24){ reject; }" comment="RFC 5737 TEST-NET-1"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==192.88.99.0/24 && dst-len >= 24){ reject; }" comment="RFC 7526 6to4 anycast relay"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==192.168.0.0/16 && dst-len >= 16){ reject; }" comment="RFC 1918 private space"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==198.18.0.0/15 && dst-len >= 15){ reject; }" comment="RFC 2544 benchmarking"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==198.51.100.0/24 && dst-len >= 24){ reject; }" comment="RFC 5737 TEST-NET-2"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==203.0.113.0/24 && dst-len >= 24){ reject; }" comment="RFC 5737 TEST-NET-3"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==224.0.0.0/4 && dst-len >= 4){ reject; }" comment="multicast"
add chain=GENERIC_PREFIX_LIST rule="if ( afi ipv4 && dst==240.0.0.0/4 && dst-len >= 4){ reject; }" comment="reserved"
```

## Junos
```
policy-statement reject-bogon-prefixes {
    term reject-bogon-prefixes-v4 {
        from {
            route-filter 0.0.0.0/8 orlonger;
            route-filter 10.0.0.0/8 orlonger;
            route-filter 100.64.0.0/10 orlonger;
            route-filter 127.0.0.0/8 orlonger;
            route-filter 169.254.0.0/16 orlonger;
            route-filter 172.16.0.0/12 orlonger;
            route-filter 192.0.2.0/24 orlonger;
            route-filter 192.88.99.0/24 orlonger;
            route-filter 192.168.0.0/16 orlonger;
            route-filter 198.18.0.0/15 orlonger;
            route-filter 198.51.100.0/24 orlonger;
            route-filter 203.0.113.0/24 orlonger;
            route-filter 224.0.0.0/4 orlonger;
            route-filter 240.0.0.0/4 orlonger;
        }
        then reject;
    }
    term reject-bogon-prefixes-v6 {
        from {
            route-filter ::/8 orlonger;
            route-filter 100::/64 orlonger;
            route-filter 2001:2::/48 orlonger;
            route-filter 2001:10::/28 orlonger;
            route-filter 2001:db8::/32 orlonger;
            route-filter 3fff::/20 orlonger;
            route-filter 2002::/16 orlonger;
            route-filter 3ffe::/16 orlonger;
            route-filter fc00::/7 orlonger;
            route-filter fe80::/10 orlonger;
            route-filter fec0::/10 orlonger;
            route-filter ff00::/8 orlonger;
        }
        then reject;
    }
}
```
## IOS-XR
```
prefix-set BOGONS_V4
  0.0.0.0/8 le 32,
  10.0.0.0/8 le 32,
  100.64.0.0/10 le 32,
  127.0.0.0/8 le 32,
  169.254.0.0/16 le 32,
  172.16.0.0/12 le 32,
  192.0.2.0/24 le 32,
  192.88.99.0/24 le 32,
  192.168.0.0/16 le 32,
  198.18.0.0/15 le 32,
  198.51.100.0/24 le 32,
  203.0.113.0/24 le 32,
  224.0.0.0/4 le 32,
  240.0.0.0/4 le 32
end-set
!
route-policy BGP_FILTER_IN
  if destination in BOGONS_V4 then
    drop
  endif
end-policy
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
            prefix-list "BOGONS_V4"
                prefix 0.0.0.0/8 longer
                prefix 10.0.0.0/8 longer
                prefix 100.64.0.0/10 longer
                prefix 127.0.0.0/8 longer
                prefix 169.254.0.0/16 longer
                prefix 172.16.0.0/12 longer
                prefix 192.0.2.0/24 longer
                prefix 192.88.99.0/24 longer
                prefix 192.168.0.0/16 longer
                prefix 198.18.0.0/15 longer
                prefix 198.51.100.0/24 longer
                prefix 203.0.113.0/24 longer
                prefix 224.0.0.0/4 longer
                prefix 240.0.0.0/4 longer
            exit
            policy-statement "BGP_FILTER_IN"
                entry 10
                    from
                        prefix-list "BOGONS_V4"
                    exit
                    action drop
                    exit
                exit
            exit
            commit
        exit
----------------------------------------------

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options prefix-list "BOGONS_V4" prefix 0.0.0.0/8 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 10.0.0.0/8 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 100.64.0.0/10 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 127.0.0.0/8 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 169.254.0.0/16 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 172.16.0.0/12 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.0.2.0/24 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.88.99.0/24 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 192.168.0.0/16 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 198.18.0.0/15 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 198.51.100.0/24 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 203.0.113.0/24 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 224.0.0.0/4 longer
/configure router policy-options prefix-list "BOGONS_V4" prefix 240.0.0.0/4 longer
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 10 from prefix-list "BOGONS_V4"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 10 action drop
/configure router policy-options commit

#
# Model-Driven CLI (MD-CLI)
#
[gl:configure policy-options]
    prefix-list "BOGONS_V4" {
        prefix 0.0.0.0/8 type longer {
        }
        prefix 10.0.0.0/8 type longer {
        }
        prefix 100.64.0.0/10 type longer {
        }
        prefix 127.0.0.0/8 type longer {
        }
        prefix 169.254.0.0/16 type longer {
        }
        prefix 172.16.0.0/12 type longer {
        }
        prefix 192.0.2.0/24 type longer {
        }
        prefix 192.88.99.0/24 type longer {
        }
        prefix 192.168.0.0/16 type longer {
        }
        prefix 198.18.0.0/15 type longer {
        }
        prefix 198.51.100.0/24 type longer {
        }
        prefix 203.0.113.0/24 type longer {
        }
        prefix 224.0.0.0/4 type longer {
        }
        prefix 240.0.0.0/4 type longer {
        }
    }
    policy-statement "BGP_FILTER_IN" {
    entry 10 {
        from {
            prefix-list ["BOGONS_V4"]
        }
        action {
            action-type reject
        }
    }

#
# Paste-friendly MD-CLI blob
#
/configure policy-options prefix-list "BOGONS_V4" { }
/configure policy-options prefix-list "BOGONS_V4" { prefix 0.0.0.0/8 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 10.0.0.0/8 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 100.64.0.0/10 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 127.0.0.0/8 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 169.254.0.0/16 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 172.16.0.0/12 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.0.2.0/24 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.88.99.0/24 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 192.168.0.0/16 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 198.18.0.0/15 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 198.51.100.0/24 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 203.0.113.0/24 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 224.0.0.0/4 type longer }
/configure policy-options prefix-list "BOGONS_V4" { prefix 240.0.0.0/4 type longer }
/configure policy-options policy-statement "BGP_FILTER_IN" { }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 10 }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 10 from prefix-list ["BOGONS_V4"]
/configure policy-options policy-statement "BGP_FILTER_IN" entry 10 action action-type reject
```
## Arista
```
ip prefix-list BOGONS_v4
   seq 1 permit 0.0.0.0/8 le 32
   seq 2 permit 10.0.0.0/8 le 32
   seq 3 permit 100.64.0.0/10 le 32
   seq 4 permit 127.0.0.0/8 le 32
   seq 5 permit 169.254.0.0/16 le 32
   seq 6 permit 172.16.0.0/12 le 32
   seq 7 permit 192.0.2.0/24 le 32
   seq 8 permit 192.88.99.0/24 le 32
   seq 9 permit 192.168.0.0/16 le 32
   seq 10 permit 198.18.0.0/15 le 32
   seq 11 permit 198.51.100.0/24 le 32
   seq 12 permit 203.0.113.0/24 le 32
   seq 13 permit 224.0.0.0/4 le 32
   seq 14 permit 240.0.0.0/4 le 32
!
route-map Import-Peer deny 20
   match ip address prefix-list BOGONS_v4
!
```

## Huawei VRP
```
ip ip-prefix prefix_Denied_Bogons_ipv4 index 20 permit 0.0.0.0 8 match-network greater-equal 8 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 30 permit 10.0.0.0 8 greater-equal 8 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 40 permit 100.64.0.0 10 greater-equal 10 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 50 permit 127.0.0.0 8 greater-equal 8 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 60 permit 169.254.0.0 16 greater-equal 16 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 70 permit 172.16.0.0 12 greater-equal 12 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 80 permit 192.0.2.0 24 greater-equal 24 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 90 permit 192.88.99.0 24 greater-equal 24 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 100 permit 192.168.0.0 16 greater-equal 16 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 110 permit 198.18.0.0 15 greater-equal 15 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 120 permit 198.51.100.0 24 greater-equal 24 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 130 permit 203.0.113.0 24 greater-equal 24 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 140 permit 224.0.0.0 4 greater-equal 4 less-equal 32
ip ip-prefix prefix_Denied_Bogons_ipv4 index 150 permit 240.0.0.0 4 greater-equal 4 less-equal 32

route-policy TRANSIT-V4-IN deny node 100
 if-match ip-prefix prefix_Denied_Bogons_ipv4
````

# Configuration Examples IPv6

## BIRD
```
define BOGON_PREFIXES = [ ::/8+,                         # RFC 4291 IPv4-compatible, loopback, et al
                          0100::/64+,                    # RFC 6666 Discard-Only
                          2001:2::/48+,                  # RFC 5180 BMWG
                          2001:10::/28+,                 # RFC 4843 ORCHID
                          2001:db8::/32+,                # RFC 3849 documentation
                          3fff::/20+,                    # draft-ietf-v6ops-rfc3849-update documentation
                          2002::/16+,                    # RFC 7526 6to4 anycast relay
                          3ffe::/16+,                    # RFC 3701 old 6bone
                          fc00::/7+,                     # RFC 4193 unique local unicast
                          fe80::/10+,                    # RFC 4291 link local unicast
                          fec0::/10+,                    # RFC 3879 old site local unicast
                          ff00::/8+                      # RFC 4291 multicast
 ];

function reject_bogon_prefixes()
prefix set bogon_prefixes;
{
    bogon_prefixes = BOGON_PREFIXES;
    if (net ~ bogon_prefixes) then {
        print "Reject: Bogon prefix: ", net, " ", bgp_path;
        reject;
    }
}

...

filter transit_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_small_prefixes();
        reject_default_route();

        honor_graceful_shutdown();
        accept;
}

filter ixp_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_transit_paths();
        reject_small_prefixes();
        reject_default_route();

        honor_graceful_shutdown();
        accept;
}

```
## FortiOS
```
config router prefix-list6
    edit "BGP_IPv6_BOGONS"
        config rule
            edit 1
                set prefix6 ::/8
                set ge 9
                unset le
            next
            edit 2
                set prefix6 100::/64
                set ge 65
                unset le
            next
            edit 3
                set prefix6 2001:2::/48
                set ge 49
                unset le
            next
            edit 4
                set prefix6 2001:10::/28
                set ge 29
                unset le
            next
            edit 5
                set prefix6 2001:db8::/32
                set ge 33
                unset le
            next
            edit 6
                set prefix6 2002::/16
                set ge 17
                unset le
            next
            edit 7
                set prefix6 3ffe::/16
                set ge 17
                unset le
            next
            edit 8
                set prefix6 fc00::/7
                set ge 8
                unset le
            next
            edit 9
                set prefix6 fe80::/10
                set ge 11
                unset le
            next
            edit 10
                set prefix6 fec0::/10
                set ge 11
                unset le
            next
            edit 11
                set prefix6 ff00::/8
                set ge 9
                unset le
            next
            edit 12
                set prefix6 3fff::/20
                set ge 21
                unset le
            next
        end
    next
end

config router route-map
    edit "BGP_FILTER_IN"
        config rule
            edit 1
                set action deny
                set match-ip6-address "IPv6_BOGONS"
            next
        end
    next
end
```
## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L111-L121)

```
deny from any prefix ::/8 prefixlen >= 8
deny from any prefix 0100::/64 prefixlen >= 64          # Discard-Only [RFC6666]
deny from any prefix 2001:2::/48 prefixlen >= 48        # BMWG [RFC5180]
deny from any prefix 2001:10::/28 prefixlen >= 28       # ORCHID [RFC4843]
deny from any prefix 2001:db8::/32 prefixlen >= 32      # docu range [RFC3849]
deny from any prefix 3fff::/20 prefixlen >= 20,         # docu range 2 [draft-ietf-v6ops-rfc3849-update]
deny from any prefix 2002::/16 prefixlen >= 16          # 6to4 anycast relay [RFC7526]
deny from any prefix 3ffe::/16 prefixlen >= 16          # old 6bone
deny from any prefix fc00::/7 prefixlen >= 7            # unique local unicast
deny from any prefix fe80::/10 prefixlen >= 10          # link local unicast
deny from any prefix fec0::/10 prefixlen >= 10          # old site local unicast
deny from any prefix ff00::/8 prefixlen >= 8            # multicast
```

## FRR (vtysh)
```
ipv6 prefix-list BOGONS_v6 deny ::/8 le 128
ipv6 prefix-list BOGONS_v6 deny 100::/64 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:2::/48 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:10::/28 le 128
ipv6 prefix-list BOGONS_v6 deny 2001:db8::/32 le 128
ipv6 prefix-list BOGONS_v6 deny 3fff::/20 le 128
ipv6 prefix-list BOGONS_v6 deny 2002::/16 le 128
ipv6 prefix-list BOGONS_v6 deny 3ffe::/16 le 128
ipv6 prefix-list BOGONS_v6 deny fc00::/7 le 128
ipv6 prefix-list BOGONS_v6 deny fe80::/10 le 128
ipv6 prefix-list BOGONS_v6 deny fec0::/10 le 128
ipv6 prefix-list BOGONS_v6 deny ff00::/8 le 128
```

## Mikrotik

### RouterOS v6
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=::/8 prefix-length=8-128 protocol=bgp action=discard comment="RFC 4291 IPv4-compatible, loopback, et al"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=0100::/64 prefix-length=64-128 protocol=bgp action=discard comment="RFC 6666 Discard-Only"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:2::/48 prefix-length=48-128 protocol=bgp action=discard comment="RFC 5180 BMWG"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:10::/28 prefix-length=28-128 protocol=bgp action=discard comment="RFC 4843 ORCHID"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2001:db8::/32 prefix-length=32-128 protocol=bgp action=discard comment="RFC 3849 documentation"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=3fff::/20 prefix-length=20-128 protocol=bgp action=discard comment="draft-ietf-v6ops-rfc3849-update documentation"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=2002::/16 prefix-length=16-128 protocol=bgp action=discard comment="RFC 7526 6to4 anycast relay"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=3ffe::/16 prefix-length=16-128 protocol=bgp action=discard comment="RFC 3701 old 6bone"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fc00::/7 prefix-length=7-128 protocol=bgp action=discard comment="RFC 4193 unique local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fe80::/10 prefix-length=10-128 protocol=bgp action=discard comment="RFC 4291 link local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=fec0::/10 prefix-length=10-128 protocol=bgp action=discard comment="RFC 3879 old site local unicast"
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix=ff00::/8 prefix-length=8-128 protocol=bgp action=discard comment="RFC 4291 multicast"
```

### RouterOS v7 
```
/routing/filter/rule
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==::/8 && dst-len >= 8 ){ reject;}" comment="RFC 4291 IPv4-compatible, loopback, et al"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==0100::/64 && dst-len >= 64 ){ reject; }" comment="RFC 6666 Discard-Only"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==2001:2::/48 && dst-len >= 48 ){ reject; }" comment="RFC 5180 BMWG"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==2001:10::/28 && dst-len >= 28 ){ reject; }" comment="RFC 4843 ORCHID"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==2001:db8::/32 && dst-len >= 32 ){ reject; }" comment="RFC 3849 documentation"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==3fff::/20 && dst-len >= 20 ){ reject; }" comment="draft-ietf-v6ops-rfc3849-update documentation"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==2002::/16 && dst-len >= 16 ){ reject; }" comment="RFC 7526 6to4 anycast relay"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==3ffe::/16 && dst-len >= 16){ reject; }" comment="RFC 3701 old 6bone"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==fc00::/7 && dst-len >=7 ){ reject; }" comment="RFC 4193 unique local unicast"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==fe80::/10 && dst-len >= 10){ reject; }" comment="RFC 4291 link local unicast"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==fec0::/10 && dst-len >= 10){ reject; }" comment="RFC 3879 old site local unicast"
add chain="GENERIC_PREFIX_LIST" rule="if ( afi ipv6 && dst==ff00::/8 && dst-len >= 8) { reject; }" comment="RFC 4291 multicast"
```

## Juniper and Cisco

Gert Doering's [ipv6-filters](https://www.space.net/~gert/RIPE/ipv6-filters.html)

## YAML from Coloclue

Coloclue's network management system [kees](https://github.com/coloclue/kees) considers these the IPv6 Bogons: [yaml file](https://github.com/coloclue/kees/blob/master/vars_example/generic.yml#L188-L274)

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
            prefix-list "BOGONS_V6"
                prefix ::/8 longer
                prefix 100::/64 longer
                prefix 2001:2::/48 longer
                prefix 2001:10::/28 longer
                prefix 2001:db8::/32 longer
                prefix 3fff::/20 longer
                prefix 2002::/16 longer
                prefix 3ffe::/16 longer
                prefix fc00::/7 longer
                prefix fe80::/10 longer
                prefix fec0::/10 longer
                prefix ff00::/8 longer
            exit
            policy-statement "BGP_FILTER_IN"
                entry 20
                    from
                        prefix-list "BOGONS_V6"
                    exit
                    action drop
                    exit
                exit
            exit
            commit
        exit

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options prefix-list "BOGONS_V6" prefix ::/8 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 100::/64 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:2::/48 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:10::/28 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2001:db8::/32 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 3fff::/20 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 2002::/16 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix 3ffe::/16 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fc00::/7 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fe80::/10 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix fec0::/10 longer
/configure router policy-options prefix-list "BOGONS_V6" prefix ff00::/8 longer
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 20 from prefix-list "BOGONS_V6"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 20 action drop
/configure router policy-options commit

#
# Model-driven CLI (MD-CLI)
#
[gl:configure policy-options]
    prefix-list "BOGONS_V6" {
        prefix ::/8 type longer {
        }
        prefix 100::/64 type longer {
        }
        prefix 2001:2::/48 type longer {
        }
        prefix 2001:10::/28 type longer {
        }
        prefix 2001:db8::/32 type longer {
        }
        prefix 3fff::/20 type longer {
        }
        prefix 2002::/16 type longer {
        }
        prefix 3ffe::/16 type longer {
        }
        prefix fc00::/7 type longer {
        }
        prefix fe80::/10 type longer {
        }
        prefix fec0::/10 type longer {
        }
        prefix ff00::/8 type longer {
        }
    }
    policy-statement "BGP_FILTER_IN" {
        entry 20 {
            from {
                prefix-list ["BOGONS_V6"]
            }
            action {
                action-type reject
            }
        }
    }

#
# Paste-friendly MD-CLI blob
#
/configure policy-options prefix-list "BOGONS_V6" { }
/configure policy-options prefix-list "BOGONS_V6" { prefix ::/8 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 100::/64 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:2::/48 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:10::/28 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2001:db8::/32 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 3fff::/20 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 2002::/16 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix 3ffe::/16 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fc00::/7 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fe80::/10 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix fec0::/10 type longer }
/configure policy-options prefix-list "BOGONS_V6" { prefix ff00::/8 type longer }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 20 }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 20 from prefix-list ["BOGONS_V6"]
/configure policy-options policy-statement "BGP_FILTER_IN" entry 20 action action-type reject
```

## Huawei VRP
```
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 20 permit 100:: 64 greater-equal 64 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 30 permit 2001:2:: 48 greater-equal 48 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 40 permit 2001:10:: 28 greater-equal 28 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 50 permit 2001:DB8:: 32 greater-equal 32 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 60 permit 2002:: 16 greater-equal 16 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 70 permit 3FFE:: 16 greater-equal 16 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 80 permit FC00:: 7 greater-equal 7 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 90 permit FE80:: 10 greater-equal 10 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 100 permit FEC0:: 10 greater-equal 10 less-equal 128
ip ipv6-prefix prefix_Denied_Bogons_ipv6 index 110 permit FF00:: 8 greater-equal 8 less-equal 128

route-policy TRANSIT-V6-IN deny node 100
 if-match ipv6 address prefix-list prefix_Denied_Bogons_ipv6
```
