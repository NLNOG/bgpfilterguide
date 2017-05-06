layout: page
title: Filtering Small Prefixes
permalink: /guides/small_prefixes.md/
---

* TOC
{:toc}

# Filter Small Prefixes

## Purpose

A basic filter set on any BGP configuration should include a filter of small prefixes.

This avoids more specific hijacks on /32's or small prefixes (targetted attacks).
Most of the leaked small prefixes that you will see on an Internet Exchange or transit feed are either incorrect leaks due to incorrect filtering or traffic engineering.

You will not miss anything as you will see the larger prefixes via the same IXP or transit feed. (the aggregated prefix)

There are some small /29 or /28 PI prefixes, but not a lot ... that get lost due to these filters ..
So be aware that you may get a question that could explain if that would happen.
Routes smaller than a /24 should not be expected to have working global routing ...

# Configuration Examples

## Juniper

policy-options {
  policy-statement bgp-import-policy {
    term nosmallprefixes {
        from {
            route-filter 0.0.0.0/0 prefix-length-range /28-/32 reject;
        }
    }
  }
}
