---
layout: page
title: BGP Filter Guides
permalink: /
---

### RPKI

Rejecting RPKI invalid BGP Routes: [reject_invalids](/guides/reject_invalids).

[RPKI FAQ](https://rpki.readthedocs.io/en/latest/about/faq.html) at the RPKI Documentation project

Locally attest well-known Trust Anchor publication point prefixes [slurm_ta](/guides/slurm_ta/).

### Bogon ASN filtering

Rejecting routes which have a Bogon ASN anywhere in the AS_PATH: [bogon_asn](/guides/bogon_asns/).

### Bogon Prefix filtering

Rejecting routes which are bogons: [bogon_prefixes](/guides/bogon_prefixes/).

### No Small Prefix filtering

Rejecting routes which have small prefixes: [No Small Prefixes](/guides/small_prefixes/).

### Filter Long AS Paths

Rejecting routes which have very long AS Paths: [No Long AS Paths](/guides/long_paths/).

### Filter Known Transit Networks in AS Paths

Rejecting routes which contain known Transit Networks in the AS Path: [No Known Transit in AS_Path](/guides/no_transit_leaks/).

### Remainder Accept term

A Remainder accept term with BGP communities and a local-preference: [Remainder accept term](/guides/remainder_accept/).

### Supporting GRACEFUL_SHUTDOWN

How to facilitate the graceful shutdown of BGP sessions: [Graceful Shutdown](/guides/graceful_shutdown/).

### Blackhole an address

How to announce an address or prefix to be blackholed using the RFC7999 BLACKHOLE community: [BLACKHOLE BGP Community](/guides/blackhole_bgp_community/).
