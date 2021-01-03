---
layout: page
title: Locally attest well-known RPKI Trust Anchor publication point prefixes
permalink: /guides/slurm_ta/
---

* TOC
{:toc}

# Locally attest well-known Trust Anchor publication point prefixes

## Purpose

[RFC 7115](https://tools.ietf.org/html/rfc7115) Section 5 says:

`
Operators should be aware that there is a trade-off in placement of
an RPKI repository in address space for which the repository's
content is authoritative.  On one hand, an operator will wish to
maximize control over the repository.  On the other hand, if there
are reachability problems to the address space, changes in the
repository to correct them may not be easily accessed by others.
`

# Solution

Operators can use the following JSON data in their SLURM file to
attempt to avoid a myriad of types of outages, by pinning the
following prefixes to specific ASNs.

`
{
  "slurmVersion": 1,
  "validationOutputFilters": {
    "locallyAddedAssertions": {
      "prefixAssertions": [
        {
          "asn": 33764,
          "prefix": "196.216.2.0/23",
          "comment": "AFRINIC"
        },
        {
          "asn": 33764,
          "prefix": "2001:42d0::/32",
          "comment": "AFRINIC"
        },
        {
          "asn": 394018,
          "prefix": "199.5.26.0/24",
          "comment": "ARIN"
        },
        {
          "asn": 393220,
          "prefix": "199.71.0.0/24",
          "comment": "ARIN"
        },
        {
          "asn": 393225,
          "prefix": "199.212.0.0/24",
          "comment": "ARIN"
        },
        {
          "asn": 394018,
          "prefix": "2001:500:a9::/48",
          "comment": "ARIN"
        },
        {
          "asn": 393220,
          "prefix": "2001:500:31::/48",
          "comment": "ARIN"
        },
        {
          "asn": 393225,
          "prefix": "2001:500:13::/48",
          "comment": "ARIN"
        },
        {
          "asn": 4608,
          "prefix": "203.119.100.0/22",
          "comment": "APNIC"
        },
        {
          "asn": 4608,
          "prefix": "2001:dd8:9::/48",
          "comment": "APNIC"
        },
        {
          "asn": 28001,
          "prefix": "200.3.12.0/22",
          "comment": "LACNIC"
        },
        {
          "asn": 28001,
          "prefix": "2001:13c7:7002::/48",
          "comment": "LACNIC"
        },
        {
          "asn": 3333,
          "prefix": "193.0.0.0/21",
          "comment": "RIPE"
        },
        {
          "asn": 3333,
          "prefix": "2001:67c:2e8::/48",
          "comment": "RIPE"
        }
      ]
    }
  }
}
`
