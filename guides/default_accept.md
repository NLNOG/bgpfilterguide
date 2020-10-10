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

## IOS-XR

```
route-policy BGP_FILTER_IN
  ...
    ...
  endif
  set community ixp-import
  set local-preference +15
  done
end-policy
```

## OpenBGPD

```
neighbor $Peer {
...
        set community <your ASN>:<peer ASN>
}

match from any community <your ASN>:<peer ASN> set localpref +15
```

## FRR (vtysh)
```
neighbor <remote_IP> remote-as <remote_ASN>
neighbor <remote_IP> sender-as-path-loop-detection
address-family <IP_version> unicast
  neighbor <remote_IP> prefix-list BOGONS_v4 in
  neighbor <remote_IP> remove-private-AS
  neighbor <remote_IP> soft-reconfiguration inbound
  neighbor <remote_IP> activate
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
            community "IXP-IMPORT"
                members "<your ASN>:<peer ASN>"
            exit
            policy-statement "BGP_FILTER_IN"
                <policy entries omitted>
                default-action accept
                    community add "IXP-IMPORT"
                    local-preference 115
                exit
            exit
            commit
        exit

#
# MD-CLI
#
[gl:configure policy-options]
community "IXP-IMPORT" {
    member "<your ASN>:<peer ASN>" { }
}
policy-statement "BGP_FILTER_IN" {
        <policy entries omitted>
        default-action {
            action-type accept
            local-preference 115
            community {
                add ["IXP-IMPORT"]
            }
        }
    }
```
