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
 - next generation
 - unicorn
 - sparkling distributed ledger
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

This document proposes an extension to the DNS Service Discovery process(?) by
integrating Service Binding (SVCB) resource records. The inclusion of SVCB
records alongside SRV records enables richer metadata exchange, enhancing the
ability to identify and select optimal connection parameters. Specifically, SVCB
records allow service instances to advertise properties such as
Application-Layer Protocol Negotiation (ALPN) identifiers and other endpoint
configuration options which streamlines client connections by providing
essential information for protocol negotiation and connection setup during the
discovery phase.

--- middle

# Introduction

DNS Service Discovery (DNS-SD) relies on a sequence of steps to enable the
discovery and connection to local network services.  This document proposes an
enhancement to the DNS Service Discovery process by integrating SVCB / HTTPS
resource records as an additional step at the end of this existing sequence to
allow clients to connect to this service instance optimally.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Motivation
Typically, the DNS-SD process begins with a client enumerating service instance
names using a PTR record query. The result of this PTR query is a list of zero
or more PTR records each pointing to a unique service instance. For example, a
query for `_foo._tcp.example.com` might return multiple PTR records, each
corresponding to a specific service instances, such as
`service1._foo._tcp.example.com` and `service2._foo._tcp.example.com`.

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
naturally in this scheme. This limitation is what the document proposes to fix
by integrating the use of SVCB / HTTPS resource records.

# Extension to Service Instance Resolution

Once the client has resolved the SRV record to identify the hostname and the port
of the service instance, but before the A and AAAA queries are issued the client
SHOULD(?) issue an SVCB query.

When querying the SVCB RR, the service is translated into a QNAME by prepending
the hostname with a label indicating the scheme prefixed with an underscore.
Since there could be more than one service instance on the same host, "Port
Prefix Naming" [RFC9460] should be used.

TODO: Why not use the service instance instead of the hostname.

For example, when the SRV record for `service1._foo._tcp.example.com` returns a
hostname `host1.example.com` and port number 8080 the client should issue a SVCB
query for `_8080._foo.host1.example.com`. The owner of the service instance could
publish this record `_8080._foo.host1.example.com 7200 IN SVCB 1 . ( alpn=h2,h3
)`. Put together, the client learns that the service instance also supports h3
(which it may prefer over h2) speeding up connection establishment. Other
SvcParamKeys such as ipv4hint, ipv6hint may also be used in conjuction.

TODO: Should any restrictions be placed on SVCB RRs? Like no alias mode?

As before, the client should then issue A and AAAA queries. However, now the
client can make an informed decisions on how best to initiate the connection to
the service instance based on the information obtained from the SVCB RRs.

TODO: How long should we wait for the SVCB answer? Resolution delay timer for 50
milliseconds?  Should we draw any parallels to happy eyeballs? The SVCB,
followed by A / AAAA?

TODO: What is the minimum set of SvcParams that we expect to see
and what do we not expect to see? ipv4hint / ipv6hint should be ok.
port is not ok.

# Alternatives to SVCB Records

We present the potential alternatives to the proposed SVCB extension, and the
limitations which make the SVCB records desriable over these alternatives.

## Use of TXT records

In the context of service discovery, while TXT records can carry arbitrary
key-value pairs, including the SvcParamKeys as defined for SVCB RRs, they lack a
standardized schema which can lead to inconsistencies and challenges in parsing
or interpreting their content. While SvcParamKeys use an IANA registry, no such
constraints exist for the keys in the TXT records which make key collision
likely.  Moreover, the TXT records are a way to provide additional information
to the service and not to advertize endpoint configuration options. SVCB records
are designed to advertise endpoint configuration options in a well-defined
format. This structured approach reduces ambiguity and also enables clients to
make informed decisions, at a host level, on selecting the most efficient
protocols without needing to parse ad hoc key values in the TXT records.

TODO: Is enumerating possible ALPNs an option?
- Why not browse _h3._foo._tcp /_h2._foo_._tcp

# Security Considerations

TODO Security

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

{:numbered="false"}
