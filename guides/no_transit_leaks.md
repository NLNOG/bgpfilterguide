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

## BIRD

```
define TRANSIT_ASNS = [ 174,                  # Cogent
                        701,                  # UUNET
                        1299,                 # Telia
                        2914,                 # NTT Ltd.
                        3257,                 # GTT Backbone
                        3320,                 # Deutsche Telekom AG (DTAG)
                        3356,                 # Level3
                        3491,                 # PCCW
                        4134,                 # Chinanet
                        5511,                 # Orange opentransit
                        6453,                 # Tata Communications
                        6461,                 # Zayo Bandwidth
                        6762,                 # Seabone / Telecom Italia
                        6830,                 # Liberty Global
                        7018 ];               # AT&T
function reject_transit_paths()
int set transit_asns;
{
        transit_asns = TRANSIT_ASNS;
        if (bgp_path ~ transit_asns) then {
                print "Reject: Transit ASNs found on IXP: ", net, " ", bgp_path;
                reject;
        }
}


...

filter transit_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_small_prefixes();
        reject_default_route();

...

        honor_graceful_shutdown();
        accept;
}

filter ixp_in {
        reject_bogon_asns();
        reject_bogon_prefixes();
        reject_long_aspaths();
        reject_transit_paths();
        reject_small_prefixes();
        reject_default_route();

...

        honor_graceful_shutdown();
        accept;
}

```

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

 as-path no-transit-import-in ".* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*";
```

## IOS-XR

```
as-path-set TRANSIT_AS
  ios-regex '.* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*'
end-set
!
route-policy BGP_FILTER_IN
  if as-path in TRANSIT_AS then
    drop
  endif
end-policy
```

## OpenBGPD

```
deny from $IXP transit-as {174,701,1299,2914,3257,3320,3356,3491,4134,5511,6453,6461,6762,6830,7018}
```

(*$IXP* represents a list of IXP peers or Route Servers

## FRR (vtysh)
```
bgp as-path access-list peerings deny .* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*
```

## Nokia SR OS

```
#
# Classic CLI
#
#--------------------------------------------------
echo "Policy Configuration"
#--------------------------------------------------
        policy-options
            begin
            as-path "TRANSIT_AS"
                expression ".* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*"
            exit
            policy-statement "BGP_FILTER_IN"
                entry 50
                    from
                        as-path "TRANSIT_AS"
                    exit
                    action drop
                    exit
                exit
            exit
            commit
        exit

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options as-path "TRANSIT_AS" expression ".* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 50 from as-path "TRANSIT_AS"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 50 action drop
/configure router policy-options commit

#
# MD-CLI
#
[gl:configure policy-options]
as-path "TRANSIT_AS" {
    expression ".* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*"
}
policy-statement "BGP_FILTER_IN" {
    entry 50 {
        from {
            as-path {
                name "TRANSIT_AS"
            }
        }
        action {
            action-type reject
        }
    }
}

#
# Paste-friendly MD-CLI blob
#
/configure policy-options as-path "TRANSIT_AS" expression ".* (174|701|1299|2914|3257|3320|3356|3491|4134|5511|6453|6461|6762|6830|7018) .*"
/configure policy-options policy-statement "BGP_FILTER_IN" { }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 50 }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 50 from }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 50 from as-path name "TRANSIT_AS"
/configure policy-options policy-statement "BGP_FILTER_IN" entry 50 action action-type reject
```
