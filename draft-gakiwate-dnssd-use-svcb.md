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

author:
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

DNS Service Discovery (DNS-SD) relies on a sequence of steps to enable the
discovery and connection to local network services. The use of Service Binding
(SVCB) resource records during the service discovery process enables service
instances to advertise properties such as Application-Layer Protocol Negotiation
(ALPN) identifiers and other endpoint configuration options. This document
describes the use of SVCB / HTTPS RRs in the DNS Service Discovery process as an
additional step to allow clients to connect to service instances optimally.

--- middle

# Introduction

This document describes the use of Service Binding (SVCB) resource records in
the DNS Service Discovery process. The use of SVCB records alongside SRV records
during service discovery enables richer metadata exchange, enhancing the ability
to identify and select optimal connection parameters.  Specifically, SVCB
records allow service instances to advertise properties such as
Application-Layer Protocol Negotiation (ALPN) identifiers and other endpoint
configuration options which streamlines client connections by providing
essential information for protocol negotiation and connection setup during the
discovery phase.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Motivation

The DNS Service Discovery, with the SRV and TXT records, provides some metadata
about the service instance. The TXT records specifically are designed to give
additional information about the service itself. The specific nature of the
additional data in TXT records, and how it is to be used, is service-dependent.
However, additional properties which are not service-specific, like supported
protocols, or privacy requirements which speed up connection establishment and
improve user privacy do not fit naturally in this scheme.

This documents describes how with the use of SVCB / HTTPS resource records
{{!SVCB=RFC9460}} in the DNS service discovery process we can support these non
service specific properties such as Application-Layer Protocol Negotiation
(ALPN) {{?ALPN=RFC7301}} identifiers and other endpoint configuration options
such as Encrypted Client Hello (ECH) {{?ECH=I-D.ietf-tls-esni}} so that a
client can select its preferred options to optimally initiate the connection
to the service instance.

# Use of SVCB with Service Instance Resolution

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

Once the client has resolved the SRV record to identify the hostname and the port
of the service instance, and along with or before the A and AAAA queries are issued,
the client can issue a query for an SVCB or HTTPS record, depending on the relevant
URI scheme the application is using. The client is expected to use "Port Prefix
Naming" {{SVCB}} to encode the port learned from the SRV response, and the scheme
from the URI being accessed by the application. If there is no notion of a URI
scheme for the application, then SVCB or HTTPS queries SHOULD NOT be made. If the
URI scheme is "http" or "https", the client will issue a query for an HTTPS record;
and if the port learned from the SRV response is the same as the default port
(443), it will leave off the port prefix.

## Example Use of SVCB RRs

Consider an example where the client application starts with a service name
`service1._foo._tcp.local` and expects to use a URI scheme of "https".
The client first queries the SRV record for `service1._foo._tcp.example.com`,
which returns a hostname `host1.example.com` and port number 8080.

~~~
service1._foo._tcp.example.com  3600  IN  SRV   (
    0  0  8080  host1.example.com )
~~~

The client application then can synthesize the URI "https://host1.example.com:8080".

The client then issues issue an HTTPS query for `_8080.host1.example.com`, and
A and AAAA queries for `host1.example.com`.

The service instance operator can publish this HTTPS record:

~~~
_8080.host1.example.com  7200  IN HTTPS  1  .  (
    alpn=h2,h3 )
~~~

All put together, the client in this case with the `alpn` values learns that the
service instance supports h3 (which the client may prefer over h2) speeding
up connection establishment with the service with its preferred protocol. Other
SvcParamKeys such as `ipv4hint` and  `ipv6hint` can also be present in the HTTPS
record.

# Security Considerations

TODO

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments

{:numbered="false"}
