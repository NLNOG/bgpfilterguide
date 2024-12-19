---
layout: page
title: Filtering Small Prefixes
permalink: /guides/small_prefixes/
---

* TOC
{:toc}

# Filter Small Prefixes

## Purpose

A basic filter set on any BGP configuration should include a filter of small prefixes.

This avoids more specific hijacks on /32's or small prefixes (targetted attacks).
Most of the leaked small prefixes that you will see on an Internet Exchange or transit feed are either incorrect leaks due to incorrect filtering or traffic engineering.

Usually you'll not miss anything as you'll see the larger prefixes via the same IXP or transit feed (the covering supernet prefix).

There are some small /29 or /28 PI prefixes, but not a lot. Such small PI prefixes get lost due to filters like these.
The shortage of IPv4 address space is insufficient of a reason to weaken sanity filters like these.
So be aware that you may get a question that could explain if that would happen.

Routes smaller than a `/24` (IPv4) or `/48` (IPv6) should not be expected to have working global routing.

# Configuration Examples

## BIRD
```
function filter_import_v4()
{
  # do not accept too short prefixes
  if (net.len > 24) then {
    print "Reject: Too small prefix: ", net, " ", bgp_path;
    reject;
  }
  
  # TODO: add all other filtering needed.
  
  # accecpt any other routes
  else accept;
}

# include the filter in each BGP session (separate for v4 and v6)
protocol bgp NAME {
  (...)
  ipv4 {
  	# perform some filtering on received routes
    import filter filter_import_v4;
  }
}
```

## FortiOS
```
config router prefix-list
    edit "IPv4_SMALL_PREFIXES"
        config rule
            edit 1
                set prefix 0.0.0.0 0.0.0.0
                set ge 25
                unset le
            next
        end
    next
end

config router prefix-list6
    edit "IPv6_SMALL_PREFIXES"
        config rule
            edit 1
                set prefix6 ::/0
                set ge 49
                unset le
            next
        end
    next
end

config router route-map
    edit "BGP_FILTER_IN"
        config rule
            edit 1
                set action deny
                set match-ip-address "IPv4_SMALL_PREFIXES"
                set match-ip6-address "IPv6_SMALL_PREFIXES"
            next
        end
    next
end
```

## Junos

```
policy-options {
  policy-statement reject_small_prefixes {
    term reject_small_prefixes_v4 {
        from {
            route-filter 0.0.0.0/0 prefix-length-range /25-/32;
        }
        then {
            reject;
        }
    }
    term reject_small_prefixes_v6 {
        from {
            route-filter ::0/0 prefix-length-range /49-/128;
        }
        then {
            reject;
        }
    }
  }
}
```


## Cisco classic IOS and IOS XE
```
ip prefix-list peerfilter seq 5 deny 0.0.0.0/0
ip prefix-list peerfilter seq 10 permit 0.0.0.0/0 ge 8 le 24

#Use a template peer-policy that you configure for each neighbor like this:
 !
 template peer-policy ixe-v4
  prefix-list peerfilter in
  maximum-prefix <number>
 exit-peer-policy
 !
router bgp <my ASN>
 !
 address-family ipv4
neighbor 192.0.2.1 inherit peer-policy ixe-v4
neighbor 192.0.2.1 activate
!
}
```

## OpenBGPD
```
deny from any inet prefixlen > 24
deny from any inet6 prefixlen > 48
```

## FRR (vtysh)
```
ip prefix-list BOGONS_v4 deny 0.0.0.0/0 ge 25 le 32
ipv6 prefix-list BOGONS_v6 deny ::/0 ge 49 le 128
```

## VyOS
```
set policy prefix-list TINY-PREFIX-V4 rule 10 action 'permit'
set policy prefix-list TINY-PREFIX-V4 rule 10 ge '25'
set policy prefix-list TINY-PREFIX-V4 rule 10 le '32'
set policy prefix-list TINY-PREFIX-V4 rule 10 prefix '0.0.0.0/0'

set policy prefix-list6 TINY-PREFIX-V6 rule 10 action 'permit'
set policy prefix-list6 TINY-PREFIX-V6 rule 10 ge '49'
set policy prefix-list6 TINY-PREFIX-V6 rule 10 le '128'
set policy prefix-list6 TINY-PREFIX-V6 rule 10 prefix '::/0'

set policy route-map INTERNET-IN rule 10 action 'deny'
set policy route-map INTERNET-IN rule 10 match ip address prefix-list 'TINY-PREFIX-V4'
set policy route-map INTERNET-IN rule 20 action 'deny'
set policy route-map INTERNET-IN rule 20 match ipv6 address prefix-list 'TINY-PREFIX-V6'
```

## Mikrotik

### RouterOS v6
This is not recommanded. Mikrotik will take a very very long time to process all those routes and has some issues with BGP.
```
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv4 prefix-length=0-7 protocol=bgp action=discard comment=""
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv4 prefix-length=25-32 protocol=bgp action=discard comment=""
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix-length=0-15 protocol=bgp action=discard comment=""
/routing filter add chain=GENERIC_PREFIX_LIST address-family=ipv6 prefix-length=49-128 protocol=bgp action=discard comment=""
```

### RouterOS v7 
```
/routing/filter/rule
add chain=GENERIC_PREFIX_LIST rule="if (afi ipv4 && dst-len > 24){ reject }"
add chain=GENERIC_PREFIX_LIST rule="if (afi ipv4 && dst-len < 7){ reject }" 
add chain=GENERIC_PREFIX_LIST rule="if (afi ipv6 && dst-len > 48){ reject }"
add chain=GENERIC_PREFIX_LIST rule="if (afi ipv6 && dst-len < 15){ reject }"
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
            prefix-list "TOO_SMALL_PREFIXES"
                prefix 0.0.0.0/0 prefix-length-range 25-32
                prefix ::/0 prefix-length-range 49-128
            exit
            policy-statement "BGP_FILTER_IN"
                entry 30
                    from
                        prefix-list "TOO_SMALL_PREFIXES"
                    exit
                    action drop
                    exit
                exit
            exit
            commit

#
# Paste-friendly Classic CLI blob
#
/configure router policy-options begin
/configure router policy-options prefix-list "TOO_SMALL_PREFIXES" prefix 0.0.0.0/0 prefix-length-range 25-32
/configure router policy-options prefix-list "TOO_SMALL_PREFIXES" prefix ::/0 prefix-length-range 49-128
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 30 from prefix-list "TOO_SMALL_PREFIXES"
/configure router policy-options policy-statement "BGP_FILTER_IN" entry 30 action drop
/configure router policy-options commit

#
# MD-CLI
#
[gl:configure policy-options]
    prefix-list "TOO_SMALL_PREFIXES" {
        prefix 0.0.0.0/0 type range {
            start-length 25
            end-length 32
        }
        prefix ::/0 type range {
            start-length 49
            end-length 128
        }
    }
    policy-statement "BGP_FILTER_IN" {
        entry 30 {
            from {
                prefix-list ["TOO_SMALL_PREFIXES"]
            }
            action {
                action-type reject
            }
        }
    }

#
# Paste-friendly MD-CLI blob
#
/configure policy-options prefix-list "TOO_SMALL_PREFIXES" { }
/configure policy-options prefix-list "TOO_SMALL_PREFIXES" prefix 0.0.0.0/0 type range start-length 25
/configure policy-options prefix-list "TOO_SMALL_PREFIXES" prefix 0.0.0.0/0 type range end-length 32
/configure policy-options prefix-list "TOO_SMALL_PREFIXES" prefix ::/0 type range start-length 49
/configure policy-options prefix-list "TOO_SMALL_PREFIXES" prefix ::/0 type range end-length 128
/configure policy-options policy-statement "BGP_FILTER_IN" { }
/configure policy-options policy-statement "BGP_FILTER_IN" { entry 30 }
/configure policy-options policy-statement "BGP_FILTER_IN" entry 30 from prefix-list ["TOO_SMALL_PREFIXES"]
/configure policy-options policy-statement "BGP_FILTER_IN" entry 30 action action-type reject
```

## Arista EOS

```
ip prefix-list TINY-PREFIX-V4
   seq 1 permit 0.0.0.0/0 ge 25 le 32
!
ipv6 prefix-list TINY-PREFIX-V6
   seq 1 permit ::/0 ge 49 le 128
!
route-map NAME-IN-V4 deny 50
    match ip address prefix-list TINY-PREFIX-V4
!
route-map NAME-IN-V6 deny 50
    match ipv6 address prefix-list TINY-PREFIX-V6
!
```

## Huawei VRP
```
ip ip-prefix default_ipv4_24 index 10 permit 0.0.0.0 0 greater-equal 8 less-equal 24

ip ipv6-prefix default_ipv6_48 index 10 permit :: 0 greater-equal 18 less-equal 48
```
