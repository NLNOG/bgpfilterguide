---
layout: page
title: Filtering Small Prefixes
permalink: /guides/small_prefixes/
---

* TOC
{:toc}

# Filter Small Prefixes

## Purpose

A basic filter set on any BGP configuration should include a filter of small prefixes.

This avoids more specific hijacks on /32's or small prefixes (targetted attacks).
Most of the leaked small prefixes that you will see on an Internet Exchange or transit feed are either incorrect leaks due to incorrect filtering or traffic engineering.

Usually you'll not miss anything as you'll see the larger prefixes via the same IXP or transit feed (the covering supernet prefix).

There are some small /29 or /28 PI prefixes, but not a lot. Such small PI prefixes get lost due to filters like these.
The shortage of IPv4 address space is insufficient of a reason to weaken sanity filters like these.
So be aware that you may get a question that could explain if that would happen.

Routes smaller than a `/24` (IPv4) or `/48` (IPv6) should not be expected to have working global routing.

# Configuration Examples

## Junos

```
policy-options {
  policy-statement bgp-import-policy {
    term reject_too_small_prefixes_v4 {
        from {
            route-filter 0.0.0.0/0 prefix-length-range /25-/32;
        }
        then {
            reject;
        }
    }
  }
}
```

## Bird
```
prefix set too_small; {
    if net.len >24 then return false;
    return true;
}
```
