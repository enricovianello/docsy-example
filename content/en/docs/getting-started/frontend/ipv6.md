---
title: "Bind StoRM services over IPv6"
linkTitle: "Bind services over IPv6"
weight: 2
noComment: true
description: >
  Example of how to configure StoRM Frontend to bind over IPv6.
---

StoRM Frontend binds on IPv6 if there's a proper `/etc/gai.conf` on host.

Download it from [here][example-gai] or copy and paste the following:

```bat
# Configuration for getaddrinfo(3).
#
# So far only configuration for the destination address sorting is needed.
# RFC 3484 governs the sorting.  But the RFC also says that system
# administrators should be able to overwrite the defaults.  This can be
# achieved here.
#
# All lines have an initial identifier specifying the option followed by
# up to two values.  Information specified in this file replaces the
# default information.  Complete absence of data of one kind causes the
# appropriate default information to be used.  The supported commands include:
#
# reload  <yes|no>
#    If set to yes, each getaddrinfo(3) call will check whether this file
#    changed and if necessary reload.  This option should not really be
#    used.  There are possible runtime problems.  The default is no.
#
# label   <mask>   <value>
#    Add another rule to the RFC 3484 label table.  See section 2.1 in
#    RFC 3484.  The default is:
#
label ::1/128       0
label ::/0          1
label 2002::/16     2
label ::/96         3
label ::ffff:0:0/96 4
label fec0::/10     5
label fc00::/7      6
label 2001:0::/32   7
label ::ffff:7f00:0001/128 8

#    This default differs from the tables given in RFC 3484 by handling
#    (now obsolete) site-local IPv6 addresses and Unique Local Addresses.
#    The reason for this difference is that these addresses are never
#    NATed while IPv4 site-local addresses most probably are.  Given
#    the precedence of IPv6 over IPv4 (see below) on machines having only
#    site-local IPv4 and IPv6 addresses a lookup for a global address would
#    see the IPv6 be preferred.  The result is a long delay because the
#    site-local IPv6 addresses cannot be used while the IPv4 address is
#    (at least for the foreseeable future) NATed.  We also treat Teredo
#    tunnels special.
#
# precedence  <mask>   <value>
#    Add another rule to the RFC 3484 precedence table.  See section 2.1
#    and 10.3 in RFC 3484.  The default is:
#
precedence  ::1/128       50
precedence  ::/0          40
precedence  2002::/16     30
precedence ::/96          20
precedence ::ffff:0:0/96  10

#
#    For sites which prefer IPv4 connections change the last line to
#
#precedence ::ffff:0:0/96  100

#
# scopev4  <mask>  <value>
#    Add another rule to the RFC 3484 scope table for IPv4 addresses.
#    By default the scope IDs described in section 3.2 in RFC 3484 are
#    used.  Changing these defaults should hardly ever be necessary.
#    The defaults are equivalent to:
#
scopev4 ::ffff:169.254.0.0/112  2
scopev4 ::ffff:127.0.0.0/104    2
scopev4 ::ffff:0.0.0.0/96       14
#
#    For sites which use site-local IPv4 addresses behind NAT there is
#    the problem that even if IPv4 addresses are preferred they do not
#    have the same scope and are therefore not sorted first.  To change
#    this use only these rules:
#
scopev4 ::ffff:169.254.0.0/112  2
scopev4 ::ffff:127.0.0.0/104    2
scopev4 ::ffff:0.0.0.0/96       14
```

Also, check your `/etc/host.conf` if `multi` is set to `on`:

```bat
[...]
multi on
[...]
```

This determines if the host is allowed to have several IP addresses, which is
usually referred to as being "multi-homed".
This flag has no effect on DNS or NIS queries.

While on CentOS the default value should be `on`, on ubuntu hosts the default
value is `off`, for example.

[example-gai]: http://www.mi.infn.it/ipv6/prefer_dual_stack_on_unspec_binding_glibc29_gai.conf
