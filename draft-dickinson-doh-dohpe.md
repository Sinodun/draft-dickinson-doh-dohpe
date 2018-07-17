%%%
    Title = "DoHPE: DoH with Privacy Enhancements"
    abbrev = "DoHPE: DoH with Privacy Enhancements"
    category = "std"
    docName= "draft-dickinson-doh-dohpe-00"
    ipr = "trust200902"
    area = "Art"
    workgroup = "doh"
    keyword = ["DNS"]
    date = 2018-07-17T00:00:00Z
    [pi]
    toc = "yes"
    compact = "yes"
    symrefs = "yes"
    sortrefs = "yes"
    subcompact = "no"
    [[author]]
    initials="S."
    surname="Dickinson"
    fullname="Sara Dickinson"
    organization = "Sinodun IT"
      [author.address]
      email = "sara@sinodun.com"
      [author.address.postal]
      streets = ["Magdalen Centre", "Oxford Science Park"]
      city = "Oxford"
      code = "OX4 4GA"
      country = 'United Kingdom'
      surname="Dickinson"
    [[author]]
    initials="W."
    surname="Toorop"
    fullname="Willem Toorop"
    organization = "NLnet Labs"
      [author.address]
      email = "willem@nlnetLabs.nl"
      [author.address.postal]
      streets = ["Science Park 400"]
      city = "Amsterdam"
      code = "1098 XH"
      country = "The Netherlands"
%%%

.# Abstract

This document describes DoHPE (DoH with Privacy Enhancements) - a privacy and
anonymity profile for DoH [@!I-D.ietf-doh-dns-over-https] clients. The profile
provides guidelines on the composition of DoH messages, designed to minimize
disclosure of identifying information.

{mainmatter}

# Introduction

The DoH protocol [@!I-D.ietf-doh-dns-over-https] is currently under development
with stated potential use cases of:

"... preventing on-path devices from interfering with DNS operations and
allowing web applications to access DNS information via existing browser APIs in
a safe way consistent with Cross Origin Resource Sharing (CORS)."

Whilst DoH has clear benefits in terms of encryption, authentication, traffic
obfuscation and ease of implementation, it introduce new privacy concerns when
compared with DNS over UDP, TCP or TLS (RFC7858) simply because it introduces a
new transport layer (HTTPS) that can contain client identifiers (e.g.
user-agent, accept-language) not present in any existing DNS transport.

The privacy considerations section of that draft state the following:

"The DoH protocol design allows applications to fully leverage the HTTP
ecosystem, including features that are not enumerated here. Utilizing the full
set of HTTP features enables DoH to be more than an HTTP tunnel, but at the cost
of opening up implementations to the full set of privacy considerations of
HTTP."

In other words a design choice was made with DoH not to add
constraints to the protocol on privacy grounds; specific choices of
functionality verses privacy are left to the implementation.

In the absence of a common standard and with many possible use cases for DoH,
different application developers are likely to implement this compromise in
different ways. Monitoring entities could then use the differences to identify
not only the device but the software originating the DNS queries.
A similar problem exists for DHCP clients which [@RFC7844] attempts to address.

The proposed profile provides a common standard that minimizes information
disclosure, including the disclosure of implementation identifiers. Whilst
fingerprinting in the TLS layer is of course also possible, minimising the
additional differences introduced by HTTPS attempts to provide a DoH profile
that has parity with DNS-over-TLS from a privacy/anonymity perspective.

One use case for this profile is a client wishing to use DoH to leverage
several of the key advantages including:

* Encryption and authentication of the connection to the server 
* Obfuscation of DNS traffic by using HTTPS on port 443 (to preventing on-path
  devices from interfering with DNS operations)
* Ability to optionally use either DNS-over-TLS or DoH servers depending on
  availability, reachability and latency

but to:

* introduce the minimum possible privacy concerns compared to e.g. DNS-over-TLS
* not 'stand-out' among a DoH client population

The cost of this decision is limitations on the HTTPS functionality available to
the client which for this use case is considered acceptable. An example of such
a client is a system stub resolver that offers multiple transports for DNS or a
Tor exit node.

One advantage of standardizing this approach is that users of DoHPE clients will
have clear privacy expectations which is not necessarily the case with general
DoH clients because the choice of balancing functionality against privacy is
implementation specific.

Whilst a simple model using HTTP CONNECT to set up RFC7858 sessions would be an
option given the privacy centric goal of this specification building on top of
DoH has several advantages including clients being able to take advantage of
media-types defined in future DoH specifications.

Two issues need to be taken into account when considering how deployable this profile is:

1. Interoperability issues in the absence of certain headers might be expected
2. Some HTTPS libraries add several headers by default and so implementors will
need to take care to override this behaviour

As with DoH, this document focuses on communication between DNS clients (such as
operating system stub resolvers) and recursive resolvers.

# Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [@!RFC8174].

# Protocol specification

The protocol specification for DoHPE is that of DoH
[@!I-D.ietf-doh-dns-over-https] with the guidelines described in the following
sections.

## Selection of DoH server

In order that all connections from a given device appear as similar as possible
DoHPE clients SHOULD by default use a system wide setting for the DoH server if one exists
and can be discovered.

## HTTPS connections

DoHPE clients SHOULD send queries over connections used solely for DoHPE
('dedicated DoHPE connections') to avoid mixing with other HTTPS traffic that
might contain HTTPS messages with client identifiers.

TODO: Consider if DoHPE clients SHOULD expose configuration options for users to select if they
are willing to fall back to using existing connections for DoHPE queries if
issues are encountered with the use of dedicated DoHPE connections.

## HTTP version

In order that all DoH connections from a device appear as similar as possible
DoHPE clients MUST use HTTP/2 [@!RFC7540] or later.

## HTTP method

TODO: Consider specifying that DoHPE clients SHOULD use either GET or POST to increase similarity?

## HTTP headers

This profile obviously requires clients to include all mandatory headers as
specified on [@RFC7540] and other applicable specifications.
[@I-D.ietf-doh-dns-over-https] also makes recommendations about the
"Accept" header which apply to this profile.

### User agent

DoHPE clients SHOULD omit the user-agent header. DoHPE clients MAY set this
header to 'DoHPE client' if interoperability issues are encountered.

### Other headers

DoHPE clients SHOULD omit all other headers.

TODO: Consider if DoHPE clients should expose configuration options so users can
choose to include specific headers if interoperability issues are encountered.
Note that such configuration could serve to allow fingerprinting.

## Client authentication

HTTP Cookies MUST NOT be sent by DoHPE clients.

## Other HTTP functionality

TODO: What else should be considered here?

## Countermeasures to Traffic Analysis

This section makes suggestions for measures that can reduce the
ability of attackers to infer information pertaining to encrypted
client queries by other means (e.g., via an analysis of encrypted
traffic size or via monitoring of the unencrypted traffic from a DNS
recursive resolver to an authoritative server).

DoHPE clients and servers SHOULD implement the following
relevant DNS extensions:

* Extension Mechanisms for DNS (EDNS(0)) padding [@RFC7830], which
  allows encrypted queries and responses to hide their size, making
  analysis of encrypted traffic harder.

Guidance on padding policies for EDNS(0) is provided in
[@I-D.ietf-dprive-padding-policy].

DoHPE clients SHOULD implement the following relevant DNS
extensions:

* Privacy election per [@RFC7871] ("Client Subnet in DNS Queries").
  If a DoHPE client does not include an edns-client-subnet EDNS0
  option with SOURCE PREFIX-LENGTH set to 0 in a query, the DNS
  server may potentially leak client address information to the
  upstream authoritative DNS servers.  A DoHPE client ought to be able
  to inform the DNS resolver that it does not want any address
  information leaked, and the DNS resolver should honor that
  request.

# IANA considerations

None

# Implementations

TODO: Add details of the next release of Stubby that will support DoHPE.

# Acknowledgements

Thanks to Stephane Bortzmeyer, Ben Schwartz and many others for initial discussions on this topic.

# Changelog

draft-dickinson-doh-dohpe-00

* Initial commit

{backmatter}


