---
layout: page
title: Filtering Long Paths
permalink: /guides/long_paths/
---

* TOC
{:toc}

# Filter Long AS Paths

## Purpose

Some networks go completely overboard on the number of pre-pending AS numbers.
And as it is a known attack from the past for AS paths longer than 256 AS numbers ..  you might want to filter on max number of AS numbers.

There are at the moment of writing this page some prefixes with about 40 ASn's in the AS_Path. So a filter of more than 50 should give no additional filtered prefixes.

A safe number on the filter would be on 100 AS's in the AS_PATH.

# Configuration Examples

## BIRD
```
function reject_long_aspaths()
{
    if ( bgp_path.len > 100 ) then {
        print "Reject: Too long AS path: ", net, " ", bgp_path;
        reject;
    }
}
```

## Junos

```
policy-options {
  policy-statement bgp-import-policy {
  term no-long-paths {
      from as-path too-many-hops;
      then reject;
     }
   }
 }

 as-path too-many-hops ".{100,}";
```

Info about the original bug report : [Link to kb article](https://kb.juniper.net/InfoCenter/index?page=content&id=JSA10418)

## IOS-XR

```
route-policy BGP_FILTER_IN
  if as-path length ge 100 then
    drop
  endif
end-policy
```

## OpenBGPD
```
deny from any max-as-len 100
```

## Mikrotik
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST bgp-as-path-length="65-4294967295" protocol=bgp action=discard comment=""
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
            policy-statement "BGP_FILTER_IN"
                entry 40
                    from
                        as-path-length 100 or-higher
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
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 40 from as-path-length 100 or-higher
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 40 action drop
/configure router policy-options commit

#
# MD-CLI
#
[gl:configure policy-options]
policy-statement "BGP_FILTER_IN" {
    entry 40 {
        from {
            as-path {
                length {
                    value 100
                    qualifier or-higher
                }
            }
        }
        action {
            action-type reject
        }
    }
}

#
# Paste-friendly MD-CLI blob
#
/configure policy-options policy-statement "BGP_FILTER_IN" { }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 40 }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 40 from }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 40 from as-path }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 40 from as-path length value 100
/configure policy-options policy-statement "BGP_FILTER_IN" entry 40 from as-path length qualifier or-higher
/configure policy-options policy-statement "BGP_FILTER_IN" entry 40 action action-type reject
```

## Arista
```
route-map BGP_FILTER_IN deny 10
   match as-path length >= 100
```

## Huawei Versatile Routing Platform (VRP)
```
route-policy TRANSIT-V4-IN deny node 120
 if-match as-path length greater-equal 100 less-equal 2047
 ```
