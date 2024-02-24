---
layout: page
title: Filtering Bogon ASNs
permalink: /guides/bogon_asns/
---

* TOC
{:toc}

# Bogon ASN filtering

Original publication: [http://as2914.net/bogon_asns/configuration_examples.txt](http://as2914.net/bogon_asns/configuration_examples.txt)

## Purpose

Private or Reserved ASNs have no place in the public DFZ. Barring these from
the DFZ helps improve accountability and dampen accidental exposure of internal
routing artifacts.

All modern devices support 4-byte ASNs. Any occurrence of "23456" in the DFZ is
a either a misconfiguration or software issue.

Rejecting all EBGP routes which contain a Bogon ASN anywhere in the `AS_PATH` is
a form of [Fail-fast](https://en.wikipedia.org/wiki/Fail-fast).

# Configuration Examples

## Junos

```
policy-options {
    as-path-group bogon-asns {
        /* RFC7607 */
        as-path zero ".* 0 .*";
        /* RFC4893 AS_TRANS */
        as-path as_trans ".* 23456 .*";
        /* RFC5398 and documentation/example ASNs */
        as-path examples1 ".* [64496-64511] .*";
        as-path examples2 ".* [65536-65551] .*";
        /* RFC6996 Private ASNs */
        as-path reserved1 ".* [64512-65534] .*";
        as-path reserved2 ".* [4200000000-4294967294] .*";
        /* RFC7300 Last 16 and 32 bit ASNs */
        as-path last16 ".* 65535 .*";
        as-path last32 ".* 4294967295 .*";
        /* IANA reserved ASNs */
        as-path iana-reserved ".* [65552-131071] .*";
    }
    policy-statement import_from_ebgp {
        term bogon-asns {
            from as-path-group bogon-asns;
            then reject;
        }
        term .....
    }
}
```

## IOS-XR

```
as-path-set bogon-asns
  # RFC7607
  ios-regex '_0_',
  # 2 to 4 byte ASN migrations
  passes-through '23456',
  # RFC5398
  passes-through '[64496..64511]',
  passes-through '[65536..65551]',
  # RFC6996
  passes-through '[64512..65534]',
  passes-through '[4200000000..4294967294]',
  # RFC7300
  passes-through '65535',
  passes-through '4294967295',
  # IANA reserved
  passes-through '[65552..131071]'
end-set

route-policy import_from_ebgp
  if as-path in bogon-asns then
    drop
  else
    ......
  endif
end-policy
```

## BIRD

```
define BOGON_ASNS = [
  0,                      # RFC 7607
  23456,                  # RFC 4893 AS_TRANS
  64496..64511,           # RFC 5398 and documentation/example ASNs
  64512..65534,           # RFC 6996 Private ASNs
  65535,                  # RFC 7300 Last 16 bit ASN
  65536..65551,           # RFC 5398 and documentation/example ASNs
  65552..131071,          # RFC IANA reserved ASNs
  4200000000..4294967294, # RFC 6996 Private ASNs
  4294967295 ];           # RFC 7300 Last 32 bit ASN

function reject_bogon_asns()
int set bogon_asns;
{
  bogon_asns = BOGON_ASNS;

  if ( bgp_path ~ bogon_asns ) then {
    print "Reject: bogon AS_PATH: ", net, " ", bgp_path;
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

## Nokia SR OS

```
bgp
    error-handling
        # RFC 7607 AS 0
        update-fault-tolerance
    exit
exit

policy-options
    begin
    as-path-group "bogon-asns"
        # RFC4893 AS_TRANS
        entry 10 expression ".* 23456 .*"
        # RFC5398 and documentation/example ASNs
        entry 15 expression ".* [64496-64511] .*"
        entry 20 expression ".* [65536-65551] .*"
        # RFC6996 private ASNs
        entry 25 expression ".* [64512-65534] .*"
        entry 30 expression ".* [4200000000-4294967294] .*"
        # RFC7300 last 16-bit and 32-bit ASNs
        entry 35 expression ".* 65535 .*"
        entry 40 expression ".* 4294967295 .*"
        # IANA reserved ASNs
        entry 45 expression ".* [65552-131071] .*"
    exit
    policy-statement "import_from_ebgp"
        entry 10
            from
                as-path-group "bogon-asns"
            exit
            action reject
        exit
    exit
    commit
exit
```

## OpenBGPD

Copied from [openbsd examples](https://github.com/openbsd/src/blob/master/etc/examples/bgpd.conf#L123-L132)

```
deny from any AS 23456                          # AS_TRANS
deny from any AS 64496 - 64511                  # Reserved for use in docs and code RFC5398
deny from any AS 64512 - 65534                  # Reserved for Private Use RFC6996
deny from any AS 65535                          # Reserved RFC7300
deny from any AS 65536 - 65551                  # Reserved for use in docs and code RFC5398
deny from any AS 65552 - 131071                 # Reserved
deny from any AS 4200000000 - 4294967294        # Reserved for Private Use RFC6996
deny from any AS 4294967295                     # Reserved RFC7300
```

## FRR (vtysh)
```
bgp as-path access-list bogon-asns deny 23456
bgp as-path access-list bogon-asns deny 64496-131071
bgp as-path access-list bogon-asns deny 4200000000-4294967295
```

## VyOS
```
set policy as-path-list BOGON-ASNS rule 10 action 'permit'
set policy as-path-list BOGON-ASNS rule 10 regex '23456'
set policy as-path-list BOGON-ASNS rule 20 action 'permit'
set policy as-path-list BOGON-ASNS rule 20 regex '64496-131071'
set policy as-path-list BOGON-ASNS rule 30 action 'permit'
set policy as-path-list BOGON-ASNS rule 30 regex '4200000000-4294967295'

set policy route-map MY-ROUTE-MAP rule 10 match as-path 'BOGON-ASNS'
```

## Mikrotik

### RouterOS v6
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_0_" protocol=bgp action=discard comment="RFC 7607"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_23456_" protocol=bgp action=discard comment="RFC 4893 AS_TRANS"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_(6449[6-9])_|_(6450[0-9])_|_(6451[0-1])_|_(6553[6-9])_|_(6554[0-9])_|_(6555[0-1])_" protocol=bgp action=discard comment="RFC 5398 and documentation/example ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_6(4(5(1[2-9]|[2-9][0-9])|[6-9][0-9][0-9])|5([0-4][0-9][0-9]|5([0-2][0-9]|3[0-5])))_" protocol=bgp action=discard comment="RFC 6996 Private ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_6555[2-9]_|_655[6-9][0-9]_|_65[6-9][0-9][0-9]_|_6[6-9][0-9][0-9][0-9]_" protocol=bgp action=discard comment="RFC IANA reserved ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_[7-9][0-9][0-9][0-9][0-9]_|_1[0-2][0-9][0-9][0-9][0-9]_|_130[0-9][0-9][0-9]_" protocol=bgp action=discard comment="RFC IANA reserved ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_1310[0-6][0-9]_|_13107[0-1]_" protocol=bgp action=discard comment="RFC IANA reserved ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_42[0-8][0-9][0-9][0-9][0-9][0-9][0-9][0-9]_" protocol=bgp action=discard comment="RFC 6996 Private ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_(429[0-3][0-9][0-9][0-9][0-9][0-9][0-9])_|_(4294[0-8][0-9][0-9][0-9][0-9][0-9])_" protocol=bgp action=discard comment="RFC 6996 Private ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_(42949[0-5][0-9][0-9][0-9][0-9])_|_(429496[0-6][0-9][0-9][0-9])_" protocol=bgp action=discard comment="RFC 6996 Private ASNs"
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path="_(4294967[0-1][0-9][0-9])_|_(42949672[0-8][0-9])_|_(429496729[0-4])_" protocol=bgp action=discard comment="RFC 6996 Private ASNs"
```

### RouterOS v7 
```
/routing/filter/num-list 
add list=BOGON-AS range=0 comment="RFC 7607"
add list=BOGON-AS range=23456 comment="RFC 4893 AS_TRANS"
add list=BOGON-AS range=64496-64511 comment="RFC 5398"
add list=BOGON-AS range=64512-65534 comment="RFC 6996"
add list=BOGON-AS range=65535 comment="RFC 7300"
add list=BOGON-AS range=65536-65551 comment="RFC 5398"
add list=BOGON-AS range=65552-131071 comment="Reserved"
add list=BOGON-AS range=4200000000-4294967294 comment="RFC 6996"
add list=BOGON-AS range=4294967294 comment="RFC 7300"

/routing/filter/rule 
add chain="GENERIC_PREFIX_LIST" rule="if (bgp-as-path [[:BOGON-AS:]]){ reject }"
```

## Huawei VRP
```
route-policy TRANSIT-IN deny node 100
 if-match as-path-filter filter_Denied_AS_Path

ip as-path-filter filter_Denied_AS_Path index 100 permit _0_
ip as-path-filter filter_Denied_AS_Path index 110 permit _23456_
ip as-path-filter filter_Denied_AS_Path index 120 permit _((6449[6-9])|(64[5-9][0-9][0-9]))_
ip as-path-filter filter_Denied_AS_Path index 130 permit _(6[5-9][0-9][0-9][0-9])_
ip as-path-filter filter_Denied_AS_Path index 140 permit _([7-9][0-9][0-9][0-9][0-9])_
ip as-path-filter filter_Denied_AS_Path index 150 permit _((1[0-2][0-9][0-9][0-9][0-9])|(130[0-9][0-9][0-9]))_
ip as-path-filter filter_Denied_AS_Path index 160 permit _((1310[0-6][0-9])|(13107[0-1]))_
ip as-path-filter filter_Denied_AS_Path index 170 permit _(42[0-8][0-9][0-9][0-9][0-9][0-9][0-9][0-9])_
ip as-path-filter filter_Denied_AS_Path index 180 permit _(429[0-3][0-9][0-9][0-9][0-9][0-9][0-9])_
ip as-path-filter filter_Denied_AS_Path index 190 permit _(4294[0-8][0-9][0-9][0-9][0-9][0-9])_
ip as-path-filter filter_Denied_AS_Path index 200 permit _((42949[0-5][0-9][0-9][0-9][0-9])|(429496[0-6][0-9][0-9][0-9]))_
ip as-path-filter filter_Denied_AS_Path index 210 permit _((4294967[0-1][0-9][0-9])|(42949672[0-8][0-9])|(429496729[0-5]))_
```


## Arista

```
ip as-path regex-mode asn

ip as-path access-list bogon-asns permit _0_ any
ip as-path access-list bogon-asns permit _23456_ any
ip as-path access-list bogon-asns permit _[64496-64511]_ any
ip as-path access-list bogon-asns permit _[65536-65551]_ any
ip as-path access-list bogon-asns permit _[64512-65534]_ any
ip as-path access-list bogon-asns permit _[4200000000-4294967294]_ any
ip as-path access-list bogon-asns permit _65535_ any
ip as-path access-list bogon-asns permit _4294967295_ any
ip as-path access-list bogon-asns permit _[65552-131071]_ any

route-map Import-Peer deny 7
   match as-path bogon-asns
!
```
