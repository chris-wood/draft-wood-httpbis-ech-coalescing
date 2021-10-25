---
title: HTTP Connection Reuse Based on TLS Encrypted ClientHello
abbrev: ECH-Based HTTP Connection Reuse
docname: draft-wood-httpbis-ech-coalescing
category: std

ipr: trust200902
area: General
workgroup: httpbis
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: C. A. Wood
    name: Christopher A. Wood
    organization: Cloudflare, Inc.
    email: caw@heapingbits.net


--- abstract

This document specifies new criteria under which HTTP/2 clients may reuse connections.
It updates {{!RFC7540}}.

--- middle

# Introduction

The HTTP/2 connection reuse policy requires is stated as follows:

   Connections that are made to an origin server, either directly or
   through a tunnel created using the CONNECT method (Section 8.3), MAY
   be reused for requests with multiple different URI authority
   components.  A connection can be reused as long as the origin server
   is authoritative (Section 10.1).  For TCP connections without TLS,
   this depends on the host having resolved to the same IP address.

   For "https" resources, connection reuse additionally depends on
   having a certificate that is valid for the host in the URI.  The
   certificate presented by the server MUST satisfy any checks that the
   client would perform when forming a new TLS connection for the host
   in the URI.

Thus, HTTPS connections require that the target resource hostname resolve
to an IP address that matches that of the candidate connection for coalescing.
This IP address match ensures that clients connect to the same service.
If a server changes IP addresses as a means of mitigating hostname-to-IP
bindings, clients are less likely to reuse connections. This can have
performance problems, due to requiring an extra connection setup phase,
as well as privacy problems.

In short, using unauthenticated IP addresses as a signal for connection
reuse is fragile. This document relaxes this requirement and introduces
another signal based on HTTPS RR answer contents {{!HTTPS-RR=I-D.ietf-dnsop-svcb-https}}.

# Conventions and Definitions

{::boilerplate bcp14}

# ECH-Based Coalescing Policy

The HTTPS RR {{!HTTPS-RR}} is a new resource record used for conveying
service information about a HTTPS endpoint to clients. Some of this information
includes, for example, TLS Encrypted ClientHello (ECH) {{!TLS-ECH=I-D.ietf-tls-esni}}
public key material. The set of hosts behind the same ECH client-facing service provider
that share the same ECH and TLS configuration information is referred to as the anonymity
set. Client-facing servers SHOULD deploy ECH in such a way so as to maximize the size of
the anonymity set where possible. This means client-facing servers should use the same
ECH configuration (ECHConfig) for as many hosts as possible.

This type of deployment model means that a given ECHConfig uniquely identifies a given
service provider. As a result, clients can use it as a signal to determine if a given
resource is hosted by the same service provider. Thus, the HTTP/2 connection reuse
policy is modified to use this signal as follows:

   Connections that are made to an origin server, either directly or
   through a tunnel created using the CONNECT method (Section 8.3), MAY
   be reused for requests with multiple different URI authority
   components.  A connection can be reused as long as the origin server
   is authoritative (Section 10.1).  For TCP connections without TLS,
   this depends on the host having resolved to the same service provider.
   Clients may implement this check in one of two ways: (1) by comparing
   for equality the resolved IP address to that of the original connection,
   or (2) by comparing for equality the "echconfig" SvcParamValue in the
   resolved HTTPS answer. For the latter case, the original connection MUST
   have successfully used the "echconfig" parameter to negotiate TLS ECH.


# HTTP/3 Reuse

The HTTP/3 connection reuse policy {{?HTTP3=I-D.ietf-quic-http}} does not require
IP addresses to match. However, as HTTP/3 is based on UDP, some clients may fall
back to HTTP/2 over TCP in networks where UDP is blocked or otherwise inoperable.
Thus, the policy described in this document only applies to HTTP/2.

# Security Considerations

Existing coalescing policies do not require IP address authentication
via DNSSEC. Thus, an adversary which can spoof A or AAAA responses can
equally spoof HTTPS responses (and ECH key values).

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
