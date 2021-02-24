---
layout: page
title: Rejecting RPKI Invalid BGP Routes
permalink: /guides/reject_invalids/
---

* TOC
{:toc}

# Reject RPKI Invalid BGP Routes

## Purpose

A complete filter for any BGP configuration should reject RPKI invalid BGP routes.

Whether a route is RPKI *invalid* or not is determined following the
[RFC 6811](https://tools.ietf.org/html/rfc6811) procedure.

It is considered *harmfull* to manipulate BGP Path Attributes (for example LOCAL_PREF or COMMUNITY) based on the RPKI Origin Validation state.
Making BGP Path Attributes dependent on RPKI Validation states introduces needless brittleness in the global routing system as explained [here](https://mailarchive.ietf.org/arch/msg/sidrops/dwQi9lgYKRVctdlMAHhtgYkzhSM/).

# Configuration Examples

## OpenBGPD

This is an example how to do Origin Validation without the RPKI-To-Router protocol.
Ensure the `rpki-client` root crontab entry is not commented out, and runs every hour.

```
# crontab -l | grep rpki
~   *   *   *   *   -ns   rpki-client && bgpctl reload
```

Import the `rpki-client` generated config and instruct `bgpd` to reject RPKI invalid routes

```
include "/var/db/rpki-client/openbgpd" # include rpki-client generated VRPs
deny quick from ebgp ovs invalid       # RFC 6811 - dont import invalids
deny quick to ebgp ovs invalid         # RFC 8893 - dont export invalids
```

## BIRD
```
function reject_invalids()
{
  if (roa_check(roas, net, bgp_path.last) == ROA_INVALID) then {
    print "Reject: RPKI Invalid: ", net, " ", bgp_path;
    reject;
  }
}
```

## Junos

Configure RTR

```
routing-options {
  autonomous-system 64511;
  validation {
    group rpki-validator {
      session 10.1.1.6 {
        local-address 10.1.1.5;
      }
    }
  }
}
```

Instruct the router to reject RPKI invalid routes, and mark `not-found` and `valid` routes with a non-transitive state.
*note*: BGP communities or other BGP Path Attributes are *not* modified based on the validation state!

```
policy-statement rpki {
  term invalid {
    from {
      protocol bgp;
        validation-database invalid;
    }
    then {
      validation-state invalid;
      reject;
    }
  }
  term valid {
    from {
      protocol bgp;
      validation-database valid;
    }
    then {
      validation-state valid;
    }
  }
  term unknown {
    from {
      protocol bgp;
      validation-database unknown;
    }
    then {
      validation-state unknown;
    }
  }
}
```

## Cisco classic IOS and IOS XE

```
route-map ebgp-in deny 1
 match rpki invalid
!
```
