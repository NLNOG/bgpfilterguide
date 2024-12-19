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

In all IOS and IOS-XE releases, [Route Origin Validation will intervene in BGP best path selection](https://www.mail-archive.com/cisco-nsp@puck.nether.net/msg68472.html), which will result in routing loops. You should use the command `bgp bestpath prefix-validate disable` to disable the integrated Route Origin Validation and do it manually in a route-map instead:

```
router bgp 64500
 bgp log-neighbor-changes
 bgp rpki server tcp 10.1.1.6
 address-family ipv4
  bgp bestpath prefix-validate disable
 address-family ipv6
  bgp bestpath prefix-validate disable
```

Match and deny based on RPKI validation state.

```
route-map ebgp-in deny 1
 match rpki invalid
!
```

An alternative and strongly discouraged way to avoid those routing loops is to use IBGP signalling (`send-community extended` and `announce rpki state` neighbor configurations). This will lead to unecessary control plane load and interdependencies.

`soft-reconfiguration inbound` should be enabled on EBGP sessions to avoid periodic BGP Router Refreshes (when VRP tables are updated).

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

## Nokia SR OS

RPKI validator can be configured via:
* The Base routing instance (example below)
* The management routing instance on OOB Ethernet port

#### Configure RTR to the RPKI validator(s)

Classic CLI configuration:
```
A:br1-nyc>config>router>origin-validation# info detail
            rpki-session 192.168.1.1
                description "RTR Server"
                no shutdown
            exit
```

MD-CLI configuration:
```
[ex:configure router "Base" origin-validation]
A:admin@br1-nyc# info
    rpki-session 192.168.1.1 {
        admin-state enable
        description "RTR Server"
    }
```

#### Drop invalid prefixes 

Dropping invalid prefixes can be done using a routing policy or in the BGP configuration.

Classic CLI routing policy configuration:
```
A:br1-nyc>config>router>policy-options# info
            policy-statement "ORIGIN_POLICY"
                entry 10
                    from
                        origin-validation-state invalid
                    exit
                    action drop
                    exit
                exit
                entry 20
                    from
                        origin-validation-state notFound
                    exit
                    action accept
                    exit
                exit                 
                entry 30
                    from
                        origin-validation-state valid
                    exit
                    action accept
                    exit
                exit
            exit
```

MD-CLI routing policy configuration:
```
[ex:configure policy-options]
A:admin@br1-nyc# info
    policy-statement "ORIGIN_POLICY" {
        entry 10 {
            from {
                origin-validation-state invalid
            }
            action {
                action-type reject
            }
        }
        entry 20 {
            from {
                origin-validation-state not-found
            }
            action {
                action-type accept
            }
        }
        entry 30 {
            from {
                origin-validation-state valid
            }
            action {
                action-type accept
            }
        }
    }
```

Classic CLI BGP configuration (group or neighbor specific):
```
A:br1-nyc>config>router>bgp# info
            best-path-selection
                origin-invalid-unusable
            exit
            group "EBGP_PEERING”
                import "ORIGIN_POLICY"
                enable-origin-validation ipv4 ipv6
            exit
            no shutdown
```


MD-CLI BGP configuration (group or neighbor specific):
```
[ex:configure router "Base" bgp]
A:admin@br1-nyc# info
    peer-ip-tracking true
    best-path-selection {
        origin-invalid-unusable true
    }
    group "EBGP_PEERING" {
        origin-validation {
            ipv4 true
            ipv6 true
        }
        import {
            policy ["ORIGIN_POLICY"]
        }
    }
 ```
 
Origin validation in a VPRN instance requires SR OS 19.7.R1 or higher.

Classic CLI VPRN BGP configuration (group or neighbor specific):
```
A:br1-nyc>config>service>vprn>bgp# info
                best-path-selection
                    origin-invalid-unusable
                exit
                group "VPRN_PEERING"
                    import "ORIGIN_POLICY"
                    enable-origin-validation ipv4 ipv6
                exit
                no shutdown
```

MD-CLI VPRN BGP configuration (group or neighbor specific):
```
[ex:configure service vprn "100" bgp]
A:admin@br1-nyc# info
    best-path-selection {
        origin-invalid-unusable true
    }
    group "VPRN_PEERING" {
        origin-validation {
            ipv4 true
            ipv6 true
        }
        import {
            policy ["ORIGIN_POLICY"]
        }
    }

 ```

UPDATE 2024-11: We have removed the ```compare-origin-validation-state true``` part under best-path-selection, as this option takes the origin validation state into account in the BGP Best Path selection process and can result in unpredicted behaviour:

_When compare-origin-validation-state is configured a new step is added to the BGP decision process after removal of invalid routes and before the comparison of local preference. The new step compares the origin validation state, so that a route with a ‛Valid’ state is preferred over a route with a ‛Not-Found’ state, and a route with a ‛Not-Found’ state is preferred over a route with an ‛Invalid’ state assuming that these routes are considered ‛usable’. The new step is skipped if the compare-origin-validation-state command is not configured._

For more information see: [Nokia Unicast Routing Protocols Guide](https://infocenter.nokia.com/public/7750SR225R1A/topic/com.nokia.Unicast_Guide/bgp_prefix_orig-d497e11185.html)

## FRR (vtysh)
First, make sure that you have `-M rpki` added to the `bgpd_options=` line of
/etc/frr/daemons
```
route-map INTERNET-IN deny 10
 match rpki invalid
exit

rpki
 rpki cache <validator_ip> 3323 preference 1
exit
```

## VyOS
```
set protocols rpki cache <validator_ip> port '3323'
set protocols rpki cache <validator_ip> preference '1'

set policy route-map INTERNET-IN rule 10 action 'deny'
set policy route-map INTERNET-IN rule 10 match rpki 'invalid'
```

## Mikrotik 

### RouterOS v7
Since RouterOS v7 MikroTik has added support for RPKI-validation. 

Configure RTR
```
/routing/rpki
add address=10.1.1.6 group=rpki-validator port=323
```

Now validate the prefixes in the EBGP ingress policy: 
```
/routing/filter/rule
add chain=GENERIC_PREFIX_LIST rule="rpki-verify rpki-validator"
add chain=GENERIC_PREFIX_LIST rule="if (rpki invalid){ reject }"
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
