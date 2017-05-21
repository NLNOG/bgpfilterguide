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
        /* RFC 4893 AS_TRANS */
        as-path as_trans ".* 23456 .*";
        /* RFC 5398 and documentation/example ASNs */
        as-path examples1 ".* [64496-64511] .*";
        as-path examples2 ".* [65536-65551] .*";
        /* RFC 6996 Private ASNs*/
        as-path reserved1 ".* [64512-65534] .*";
        as-path reserved2 ".* [4200000000-4294967294] .*";
        /* RFC 6996 Last 16 and 32 bit ASNs */
        as-path last16 ".* 65535 .*";
        as-path last32 ".* 4294967295 .*";
        /* RFC IANA reserved ASNs*/
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
define BOGON_ASNS = [ 0,                      # RFC 7607
                      23456,                  # RFC 4893 AS_TRANS
                      64496..64511,           # RFC 5398 and documentation/example ASNs
                      64512..65534,           # RFC 6996 Private ASNs
                      65535,                  # RFC 6996 Last 16 bit ASN
                      65536..65551,           # RFC 5398 and documentation/example ASNs
                      65552..131071,          # RFC IANA reserved ASNs
                      4200000000..4294967294, # RFC 6996 Private ASNs
                      4294967295 ];           # RFC 6996 Last 32 bit ASN

function ebgp_import()
int set bogon_asns;
{
    # ignore bogon AS_PATHs
    bogon_asns = BOGON_ASNS;
    if ( bgp_path ~ bogon_asns ) then {
        print "Reject: bogon AS_PATH: ", net, " ", bgp_path;
        reject;
    }
    .......
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
        # RFC 4893 AS_TRANS
        entry 10 expression ".* 23456 .*"
        # RFC 5398 and documentation/example ASNs
        entry 15 expression ".* [64496-64511] .*"
        entry 20 expression ".* [65536-65551] .*"
        # RFC 6996 private ASNs
        entry 25 expression ".* [64512-65534] .*"
        entry 30 expression ".* [4200000000-4294967294] .*"
        RFC 6996 last 16-bit and 32-bit ASNs
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
