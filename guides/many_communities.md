---
layout: page
title: Strip high number of BGP Communities
permalink: /guides/many_communities/
---

* TOC
{:toc}

# Strip high number of BGP Communities

## Purpose

BGP Communities offer much useful information, but they also take up space in router memory.
Some networks or route servers attach an excessive number of communities that offer no benefit or functionality outside of their own ASN.

If you wish to put a limit on this you could strip regular, extended or large communities from a route when their number exceeds 100.

# Configuration Examples

## OpenBGPD

```
match from any max-communities 100 set { community delete *:* }
match from any max-ext-communities 100 set { ext-community delete * * }
match from any max-large-communities 100 set { large-community delete *:*:* }
```

## BIRD

```
function strip_too_many_communities() {
    if ( ( bgp_community.len + bgp_ext_community.len + bgp_large_community.len ) >= 100 ) then {
        bgp_community.empty;
        bgp_ext_community.empty;
        bgp_large_community.empty;
    }
}
```

## Junos

```
policy-options {
    policy-statement bgp-import-policy {
        term no-many-communities {
        from community-count 100 orhigher;
        then {
            community delete ALL-COMMUNITIES;
        }
    }
    community ALL-COMMUNITIES members [ *:* origin:*:* large:*:*:* ];
}
```

## Arista

```
route-map BGP_FILTER_IN permit 90
    match community instances >= 100
    set community none
!
```
