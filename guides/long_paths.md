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

## Bird
```
prefix set unwanted; {
    if ( bgp_path.len > 100 ) then return false;
    return true;
}
```


## OpenBGPD
```
deny from any max-as-len 100
```
