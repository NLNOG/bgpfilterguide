---
layout: page
title: Filtering Bogon ASNs
permalink: /guides/bogon_asns/
---

### Bogon ASN filtering

Original publication: [http://as2914.net/bogon_asns/configuration_examples.txt](http://as2914.net/bogon_asns/configuration_examples.txt)

## Juniper

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
   :
 }
```
