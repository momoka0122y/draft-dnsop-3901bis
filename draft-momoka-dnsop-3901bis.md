---
title: "DNS IPv6 Transport Operational Guidelines"
abbrev: 3901bis
docname: draft-momoka-dnsop-3901bis-latest
category: info

ipr: trust200902
area: Ops
workgroup: dnsop
keyword:
  - DNS
  - IPv6
submissionType:
  IETF
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    fullname:
      :: 山本 桃歌
      ascii: Momoka Yamamoto
    organization: The University of Tokyo/WIDE Project
    email: momoka.my6@gmail.com




normative:


informative:



--- abstract

This memo provides guidelines and documnets Best Current Practice for operating
authoritative and recursive DNS servers given that queries and responses are
carried in a mixed environment of IPv4 and IPv6 networks. It expands beyond
RFC3901 in so far that it now considers the reality of progressing IPv4 exhaustion,
which will make IPv6 only resolvers necessary in the long-term.

--- middle

# Introduction

Despite IPv6 being first discussed in the mid 1990s {{?RFC1883}}, consistent deployment thorughout the whole Internet has not yet been accomplished.
Hence, today, the Internet is a mixture of IPv4, dual-stack (networks connected via both IP versions), and IPv6 networks.

This creates a
complex landscape where authoritative name servers might be accessible only
via specific network protocols. At the same time, DNS resolvers may only be able to access the Internet via either IPv4 or IPv6. This poses a challenge for such resolvers,
as they may encounter names for which queries have to be sent to authoritative name servers with which they do not share an IP version
during the name resolution process.

In this document, we discuss:

- IP version related namespace fragmentation and best-practices for avoiding it.
- Guidelines for configuring authorative name servers for zones.
- Guidelines for operating recursive DNS resolvers.

While transitional technologies and dual-stack setups may mitigate some of the issues of DNS reolution in a mixed protocol-version Internet,
making DNS
data accessible over both IPv4 and IPv6 is the most robust and flexible
approach, as it allows resolvers to reach the information they need without
requiring intermediary translation or forwarding services which may introduce additional failure cases.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 {{!RFC2119}} {{?RFC8174}} when, and only when, they appear in all
capitals, as shown here.

# Terminology

This document uses DNS terminology as described in {{?RFC8499}}.
Furthermore, the following terms are used with a defined meaning:

- "IPv4 name server": This refers to a name server providing DNS
  services reachable via IPv4. It does not imply anything about what DNS data is
  served, but requires DNS queries to be received and answered over IPv4.
- "IPv6 name server": This refers to a name server providing DNS
  services reachable via IPv6. It does not imply anything about what DNS data is
  served, but requires DNS queries to be received and answered over IPv6.
- "dual-stack name server": A name server that is both an "IPv4 name server"
  and also an "IPv6 name server".

# Name Space Fragmentation

A resolver that tries to look up a name starts out at the root, and
follows referrals until it is referred to a name server that is
authoritative for the name.  If somewhere down the chain of referrals
it is referred to a name server that is, based on the referral, only accessible over a
transport which the resolver cannot use, the resolver is unable to
continue DNS resolution.

If this occurs, the DNS has, effectively, fragmented based on the resolver's and authoritative's mismatching IP version support.

In a mixed IP Internet, this fragmentation can be due to DNS zones being consistently configured to only support either IPv4 or IPv6, or due to misconfigurations leading to a situation where a zone is not resolvable by either IPv4 or IPv6 only resolvers due to a misconfiguration.
The latter cases are often hard to identify, as the impact of misconfigurations for only one IP version (IPv4 or IPv6) may be hidden in a dual-stack setting.
In the worst case, a specific name may only be resolvable via dual-stack enabled resolvers.


## Misconfigurations Causing IP Version Related Name Space Fragmentation

Even when an administrator thinks they have enabled support for a specific IP version on their authoritative name server, various misconfigurations may break the DNS delegation chain of a zone for that protocol and prevent any of its records from resolving for clients only supporting that IP version.
These misconfigurations can be kept hidden if most clients can successfully fall back to the other IP version.
As such, these issues are more common for IPv6 resolution related name space fragmentation.

The following name related misconfigurations can cause broken delegation for one IP version:

1. No A/AAAA records for NS names: If none of the NS records for a zone in their parent zone have associated A or AAAA records, while holding the inverse record, resolution via the concerned IP version is not possible.
2. Missing GLUE: If the name from an NS record for a zone is in-bailiwick, i.e., the name is within the zone or below, a parent zone must contain an IPv4 and IPv6 GLUE record, i.e., a parent must serve the corresponding A or AAAA record(s) as ADDITIONAL data when returning the NS record in the ANSWER section.
3. No A/AAAA record for in-bailiwick NS : If an NS record of a zone points to a name that is in-bailiwick but the name lacks corresponding A or AAAA record(s) in its zone, resolution via the concerned IP version will fail even if the parent provides GLUE, when the recursive server validates the delegation path.
4. Zone of out-of-bailiwick NSes not resolving: If an NS record of a zone is out-of-bailiwick, the corresponding zone must be resolvable via the IP version in question as well. It is insufficient if the name pointed to by the NS record has an associated A or AAAA record correspondingly.
5. Parent zone not resolvable via one IP version: For a zone to be resolvable via an IP version the parent zones up to the root zone must be resolvable via that IP version as well. Any zone not resolvable via the concerned IP version breaks the delegation chain for all its children.

The above misconfigurations are not mutually exclusive.

Furthermore, any of the misconfigurations above may also materialize not via a missing RR but via an RR providing the IP address of a nameserver that is not configured to answer queries via that IP version.

## Reasons for Intentional IP Version Related Name Space Fragmentation

Intentional IP related name space fragmentation occurs if an operator consciously decides to not deploy IPv4 or IPv6 for a part of the resolution chain.
Most commonly, this is realized by consciously implementing points 1-3 from the above list.

# Policy Based Avoidance of Name Space Fragmentation

Today there are a lot DNS "zones" on the public Internet that
are available over IPv6 transport, and the numbers are growing year by year.
However, there are still a lot of "zones" that are not yet IPv6 capable so
resolvers still need IPv4 connectivity to resolve all domain names.

## Guidelines for DNS Zone Configuration

Having zones served only by IPv6-only name server would not be
a good development either, since this will fragment the previously
unfragmented IPv4 name space and there are strong reasons to find a
mechanism to avoid it.

The recommended approach to maintain name space continuity is to use
administrative policies, as described in the next section.


## Guidelines for Authoritative Name Servers

IPv4 adoption:
Every authoritative DNS zone SHOULD be served by at least one IPv4-reachable authoritative name server to maintain name space continuity.

IPv6 Adoption:
Authoritative name servers SHOULD aim to be accessible via IPv6 to ensure service reliability for IPv6-only resolvers.
It is recognized that some networks have not yet adopted IPv6 connectivity, making this a challenging guideline to implement universally. However, for networks that do have IPv6 capability but have not yet configured their authoritative name servers to run on IPv6, it is recommended to do so to prevent any disruption in services for IPv6-only resolvers.

Consistency:
Both IPv4 and IPv6 transports should serve identical DNS data to ensure a consistent resolution experience across different network types.

Note: zone validation processes SHOULD ensure that there is at least
one IPv4 address record available for the name servers of any child
delegations within the zone.


## Guidelines for DNS Resolvers

Every iterative name server SHOULD be either IPv4-only or dual stack.

The zones a IPv6-only iterative resolvers can resolve are growing but as they cannot fully resolve all zones, they are not advisable at the moment.

While IPv6-only iterative resolvers are not recommended, configurations may be designed where such resolvers forward queries to a set of dual-stack recursive name servers that perform the actual recursive queries.

# Security Considerations

The guidelines described in this memo introduce no new security
considerations into the DNS protocol or associated operational
scenarios.

# IANA Considerations

This document does not require IANA actions.


# Acknowledgments
{:numbered="false"}

TODO: acknowledge people.

Thank you for reading this draft.
