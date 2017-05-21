---
layout: page
title: Filtering Known Transit Networks
permalink: /guides/no_transit_leaks/
---

* TOC
{:toc}

# Filter Known Transit Networks in AS Paths

## Purpose

Across an IXP, Tier 2 and Tier 3 networks should not be announcing prefixes with a transit network in the AS Path which is 'probably' not one of their customers. And you should also for the same reason, not accept any of them via one of your customers if they are not in the business of providing transit to companies like Level3, NTT or Telia.

There was a presentation at Nanog by Job Snijders that explains more about the topic. [Presentation in PDF](https://www.nanog.org/sites/default/files/Snijders_Everyday_Practical_Bgp.pdf)

Be aware that you need to manually check the prefix list as you could peer with for instance Microsoft of other parties on the list..
So you need to do a quick sanity check on the AS numbers to fit your need.  

Target import policy :  customers and IXP peering

# Configuration Examples

## Junos

```
policy-options {
  policy-statement bgp-import-policy {
    term no-transit-leaks {
        from as-path no-transit-import-in;
        then reject;
    }
   }
 }

 as-path no-transit-import-in ".* (174|209|701|702|1239|1299|2914|3257|3320|3356|3549|3561|4134|5511|6453|6461|6762|7018) .*";
```

## OpenBGPD

```
deny from $IXP transit-as {174,209,701,702,1239,1299,2914,3257,3320,3356,3549,3561,4134,5511,6453,6461,6762,7018}
```

(*$IXP* represents a list of IXP peers or Route Servers)
