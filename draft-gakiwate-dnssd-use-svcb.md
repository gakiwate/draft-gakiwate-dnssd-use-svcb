---
title: "Use SVCB with DNS Service Discovery"
category: info

docname: draft-gakiwate-dnssd-use-svcb-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "Extensions for Scalable DNS Service Discovery"
keyword:
venue:
  group: "Extensions for Scalable DNS Service Discovery"
  type: ""
  mail: "dnssd@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnssd/"
  github: "gakiwate/draft-gakiwate-dnssd-use-svcb"
  latest: "https://gakiwate.github.io/draft-gakiwate-dnssd-use-svcb/draft-gakiwate-dnssd-use-svcb.html"

author
 -
    fullname: Gautam Akiwate
    organization: Apple Inc
    email: "gakiwate@apple.com"
 -
    fullname: Tommy Pauly
    organization: Apple Inc
    email: "tpauly@apple.com"

normative:

informative:

--- abstract

This document describes the use of Service Binding (SVCB) resource records in
the DNS Service Discovery process. The use of SVCB records alongside SRV records
during service discovery enables richer metadata exchange, enhancing the ability
to identify and select optimal connection parameters.  Specifically, SVCB
records allow service instances to advertise properties such as
Application-Layer Protocol Negotiation (ALPN) identifiers and other endpoint
configuration options which streamlines client connections by providing
essential information for protocol negotiation and connection setup during the
discovery phase.

--- middle

# Introduction

DNS Service Discovery (DNS-SD) relies on a sequence of steps to enable the
discovery and connection to local network services.  This document describes the
use of SVCB / HTTPS RRs in the DNS Service Discovery process as an additional
step to allow clients to connect to this service instance optimally.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Motivation

Typically, the DNS-SD {{!DNSSD=RFC6763}} process begins with a client
enumerating service instance names using a PTR record query. The result of this
PTR query is a list of zero or more PTR records each pointing to a unique
service instance. For example, a query for `_foo._tcp.example.com` might return
multiple PTR records, each corresponding to a specific service instances, such
as `service1._foo._tcp.example.com` and `service2._foo._tcp.example.com`.

Once the client identifies the exact service instance it wants to connect to,
the client issues an SRV query for the service instance. The SRV record for the
service instance includes the hostname of the service instance, the port number
on which the service is listening, and priority and weight fields for load
balancing or failover. For example, querying the SRV record for
`service1._foo._tcp.example.com` might return a hostname like `host1.example.com`
and a port number, such as `8080`.

After resolving the SRV record, the client issues A and AAAA queries to map the
hostname in the SRV record to an IP address. For example, querying for
`host1.example.com` might return an A record with the IPv4 address `192.0.2.3` or an
AAAA record with the IPv6 address `2001:db8::1`. Once the IP address is obtained,
the client can initiate a connection to the desired service instance using the
port number specified in the SRV record.

However, this sequence provides only minimal metadata about the service itself
which is why clients also query for TXT records (alongside SRV records) for the
service instance. These TXT records give additional information about the
service itself. The specific nature of the additional data in TXT records, and
how it is to be used, is service-dependent. However, additional properties which
are not service-specific, like supported protocols, or privacy requirements
which speed up connection establishment and improve user privacy do not fit
naturally in this scheme. This documents describes how this limitation can be
overcome by integrating the use of SVCB / HTTPS resource records {{!SVCB=RFC9460}}
in the DNS service discovery process.

# Use of SVCB with Service Instance Resolution

Once the client has resolved the SRV record to identify the hostname and the port
of the service instance, but before the A and AAAA queries are issued the client
SHOULD issue an SVCB query. The service is translated into a QNAME by prepending
the hostname with a label indicating the scheme prefixed with an underscore.
Since there could be more than one service instance on the same host, "Port
Prefix Naming" {{SVCB}} should be used.

## Example Use

For example, if the SRV record for `service1._foo._tcp.example.com` returns a
hostname `host1.example.com` and port number 8080.

```
   service1._foo._tcp.example.com  3600  IN  SRV   0  0  8080  host1.example.com
```

In this case, the client SHOULD issue a SVCB query for `_8080._foo.host1.example.com`.

The service instance operator can publish this SVCB record:

```
   _8080._foo.host1.example.com  7200  IN SVCB  1  .  ( alpn=h2,h3 )
```

All put together, the client in this case learns that the service instance also
supports h3 (which the client may prefer over h2) speeding up connection
establishment with the service with its preferred protocol. Other SvcParamKeys
such as ipv4hint, ipv6hint may also be used in conjuction.

As before, the client should then issue A and AAAA queries. However, now the
client can make an informed decisions on how best to initiate the connection to
the service instance based on the information obtained from the SVCB RRs.

# Security Considerations


# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

{:numbered="false"}
