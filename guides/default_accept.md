---
layout: page
title: Remainder Accept term
permalink: /guides/remainder_accept/
---

* TOC
{:toc}

# A remainder accept term

## Purpose

After all those nice filters of what you don't want .. you may also want to accept some prefixes.

If you create a community in your ingress accept policy, you can always see where a specific prefix originated from.
You should do this as a good practice for all your bgp-import-policies. Customers, transits and IXP peerings.

If you tag all customer prefixes with BGP communities, it will also allow you to use those same communities in export policies towards your transits and bgp peers.

The local preference line it to give peering prefixes a better priority towards transit learned prefixes.

Target import policy : any bgp import policy

# Configuration Examples

## Junos

```
policy-options {
  policy-statement bgp-import-policy {

...
...

  term remainder {
      then {
          local-preference add 15;
          community add ixp-import;
          accept;
      }
   }
 }

community ixp-import members <your ASN>:<peer ASN>;
```
