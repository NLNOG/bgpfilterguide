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

Whether a route is RPKI *invalid* or not is determined following [RFC 6811](https://tools.ietf.org/html/rfc6811) and [RFC 8893](https://tools.ietf.org/html/rfc8893).

It is considered *harmful* to manipulate BGP Path Attributes (for example LOCAL_PREF or COMMUNITY) based on the RPKI Origin Validation state.
Making BGP Path Attributes dependent on RPKI Validation states introduces needless brittleness in the global routing system as explained [here](https://mailarchive.ietf.org/arch/msg/sidrops/dwQi9lgYKRVctdlMAHhtgYkzhSM/).
Additionally, the use of [RFC 8097](https://tools.ietf.org/html/rfc8097) is STRONGLY ABSOLUTELY NOT RECOMMENDED.
RFC 8097 has caused issues for multi-vendor network operators.

The RPKI documentation project lists various [Relaying Party software](https://rpki.readthedocs.io/en/latest/tools.html) packages.

# Configuration Examples

## OpenBGPD on OpenBSD

This is an example how to do Origin Validation without the RPKI-To-Router protocol.
Ensure the `rpki-client` root crontab entry is enabled and runs every hour.

```
# crontab -l | grep rpki
~   *   *   *   *   -ns   rpki-client && bgpctl reload
```

Import the `rpki-client` generated config and instruct `bgpd` to reject RPKI invalid routes

```
include "/var/db/rpki-client/openbgpd" # consume VRPs from rpki-client
deny quick from ebgp ovs invalid       # dont import invalids
deny quick to ebgp ovs invalid         # dont export invalids
```

It is REALLY NOT recommended to make set or modify any BGP Path Attributes based on the Origin Validation state (keyword: `ovs`).

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

Instruct the router to reject RPKI invalid routes, and also mark `not-found` and `valid` routes with a non-transitive state.
*note*: BGP communities or other BGP Path Attributes *REALLY MUST NOT* be modified based on the validation state!

```
policy-statement rpki {
  term reject_invalid {
    from {
      protocol bgp;
        validation-database invalid;
    }
    then {
      validation-state invalid;
      reject;
    }
  }
  term mark_valid {
    from {
      protocol bgp;
      validation-database valid;
    }
    then {
      validation-state valid;
      next policy;
    }
  }
  then {
    validation-state unknown;
    next policy;
  }
}
```

## Cisco classic IOS and IOS XE

Configure RTR

```
# show running-config | begin bgp

router bgp 64500
 bgp log-neighbor-changes
 bgp rpki server tcp 10.1.1.6
```

Match and deny based on RPKI validation state.

```
route-map ebgp-in deny 1
 match rpki invalid
!
```

## Cisco IOS-XR

*note:* ABSOLUTELY DO NOT configure `bgp origin-as validation signal ibgp` (see above note above about RFC8097).
Enabling validation signaling causes unnecesary BGP protocol chatter in the routing system.

Configure RTR and enable RPKI for each address family
```
router bgp 1
 bgp bestpath origin-as allow invalid
 address-family ipv4 unicast
   bgp origin-as validation enable
 address-family ipv6 unicast
   bgp origin-as validation enable
 rpki server 10.1.1.6
```
Match and deny based on RPKI validation state
```
route-policy rpki-validate
  if validation-state is invalid then
    drop
  endif
end-policy
```
Apply policy in policy chain where needed
```
route-policy ebgp-in
  apply rpki-validate
  <rest of policy>
```

Don't forget to enable `soft-reconfiguration inbound always` for each EBGP neighbor for each address-family! It is too bad this does not happen by default.

## BIRD

BIRD 2.0 supports RTR. However, the current implementation does not perform an automatic revalidation of routes upon receipt of new ROAs.

> The RPKI-RTR protocol receives and maintains a set of ROAs from a cache server (also called validator). You can validate routes (RFC 6483) using function `roa_check()` in filter and set it as import filter at the BGP protocol. BIRD should re-validate all of affected routes after RPKI update by RFC 6811, but we don't support it yet! You can use a BIRD's client command `reload in bgp_protocol_name` for manual call of revalidation of all routes.
(https://bird.network.cz/?get_doc&v=20&f=bird-6.html#ss6.13)

The [rtrsub](https://github.com/job/rtrsub) utility can be used to generate static ROA tables for BIRD 1.6.

Set up RTR as following:

```
roa4 table r4;
roa6 table r6;

protocol rpki {
  roa4 {
    table r4;
  };
  roa6 {
    table r6;
  };
  remote "10.1.1.6" port 323;
}
```

Define a function which returns `false` when a BGP route is RPKI invalid.

*Note:* REALLY DONT store the validation state inside a `bgp_community` or `bgp_large_community` or `bgp_ext_community` variables.
It can cause CPU & memory overload resulting in convergence performance issues.

```
function check_rpki_rov()
{
  if (roa_check(r4, net, bgp_path.last) = ROA_INVALID ||
      roa_check(r6, net, bgp_path.last) = ROA_INVALID) then {
        return false;
  }

  return true;
}

# Filter applied to EBGP session definde in protocol
filter ebgp_inbound
{
 if check_rpki_rov() then {
   reject;
 }
 accept;
}

# example protocol definition with filter applied
protocol bgp PEER1 from PEERS_TEMPLATE {
  description "PEER DESCRIPTION";
  local 192.0.2.1 as 64500;
  neighbor 192.0.2.2 as 64500;
  ipv4 {
    filter filter ebgp_inbound;
  };
  ipv6 {
    filter filter ebgp_inbound;
  };
}
```

## Nokia SR-OS

RPKI validator can be configured within the:
* Base router
* VPRN (requires SR-OS 19.7 or higher)
* Via OOB (example below)

Configure RTR to the RPKI validator(s) via OOB: 
```
A:SR-OS# /configure router "management" 
A:SR-OS>config>router# info 
----------------------------------------------
#--------------------------------------------------
echo "Origin Validation Configuration"
#--------------------------------------------------
        origin-validation
            rpki-session 10.1.1.6
                description "rtr server"
                port 323
                no shutdown
            exit
        exit
----------------------------------------------
A:SR-OS>config>router# 
```

Dropping invalid prefixes can be done by policy or configuration:

Match and drop based on RPKI invalids based on policy
```
policy-statement "rpki-rov"
    entry 100
        description "Drop RPKI invalid prefixes"
        from
            origin-validation-state invalid
        exit
        action drop
        exit
    exit
```

Match and drop based on RPKI invalids based on standard configuration statements (BGP group or neighbor specific) 

Base:
```
A:SR-OS>edit-cfg# /configure router bgp group <group-name> enable-origin-validation ipv4 ipv6
A:SR-OS>edit-cfg# /configure router bgp group <group-name> neighbor 192.0.2.2 enable-origin-validation ipv4
A:SR-OS>edit-cfg# /configure router bgp group <group-name> neighbor 2001:db8:ffff::2 enable-origin-validation ipv6
A:SR-OS>edit-cfg# /configure router bgp best-path-selection origin-invalid-unusable
```

VPRN:
```
A:SR-OS>edit-cfg# /configure service vprn <X> bgp group <group-name> enable-origin-validation ipv4 ipv6
A:SR-OS>edit-cfg# /configure service vprn <X> bgp group <group-name> neighbor 192.0.2.2 enable-origin-validation ipv4
A:SR-OS>edit-cfg# /configure service vprn <X> bgp group <group-name> neighbor 2001:db8:ffff::2 enable-origin-validation ipv6
A:SR-OS>edit-cfg# /configure service vprn <X> bgp best-path-selection origin-invalid-unusable
```

## Huawei VRP

Configure RTR and enable RPKI.

```
rpki
  session 10.1.1.6
  tcp port 323
#
bgp 65535
  prefix origin-validation enable
  bestroute origin-as-validation
```

The BGP best path selection process now ignores the routes with validation result Invalid during route selection.

## Arista EOS

Set up RTR to RPKI Validator Cache:

```
router bgp 64500
  # configure RTR
  rpki cache 10.1.1.6
    host 10.1.1.6
    local-interface Loopback0
  !
  # enable origin validation
  rpki origin-validation
    ebgp local
!
```

Now match in EBGP ingress policy and reject RPKI invalid BGP routes:

```
route-map ebgp-in deny 1
   match origin-as validity invalid
!
```
