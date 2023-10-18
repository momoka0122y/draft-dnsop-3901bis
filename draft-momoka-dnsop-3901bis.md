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
carried in a mixed environment of IPv4 and IPv6 networks.

--- middle

# Introduction

Despite IPv6 being first discussed in the mid 1990s {{RFC1883}}, consistent deployment thorughout the whole Internet has not yet been accomplished.
Hence, today, the Internet is a mixture of IPv4, dual-stack (networks connected via both IP protocol versions), and IPv6 networks.

This creates a
complex landscape where authoritative name servers might be accessible only
via specific network protocols. At the same time, DNS resolvers may only be able to access the Internet via either IPv4 or IPv6. This poses a challenge for such resolvers,
as they may encounter names for which queries have to be sent to authoritative name servers with which they do not share an IP protocol version
during the name resolution process.

In this document, we discuss:
- IP protocol version related namespace fragmentation and best-practices for avoiding it.
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

# Name Space Fragmentation: following the referral chain

A resolver that tries to look up a name starts out at the root, and
follows referrals until it is referred to a name server that is
authoritative for the name.  If somewhere down the chain of referrals
it is referred to a name server that is only accessible over a
transport which the resolver cannot use, the resolver is unable to
finish the task.

With all DNS data only available over IPv4 transport everything is
simple.  IPv4 resolvers can use the intended mechanism of following
referrals from the root and down while IPv6 resolvers have to work
through a "translator", i.e., they have to use a recursive name
server on a so-called "dual stack" host as a "forwarder" since they
cannot access the DNS data directly.

With all DNS data only available over IPv6 transport everything would
be equally simple, with the exception of IPv4 recursive name servers
having to switch to a forwarding configuration.

The transition from IPv4 only to a mixture of IPv4 and IPv6, with
three categories of DNS data depending on whether the information is
available only over IPv4 transport, only over IPv6 or both.

Having DNS data available on both transports is the optimal
situation.




# Policy Based Avoidance of Name Space Fragmentation

Today there are a lot DNS "zones" on the public Internet that
are available over IPv6 transport, and the numbers are growing year by year.
However, there are still a lot of "zones" that are not yet IPv6 capable so
resolvers still need IPv4 connectivity to resolve all domain names.

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

### Reasons for Broken IPv6 Delegation

Even when an administrator thinks they have enabled IPv6 on their authoritative name server misconfiguration specific to IPv6 may break the DNS delegation chain of a zone and prevent any of its records from resolving in an IPv6-only scenario. These misconfigurations can be kept for a long time as resolvers can fallback to IPv4.

The following misconfigurations can cause broken IPv6-delegation in an IPv6-only setting:

No AAAA records for NS names: If none of the NS records for a zone in their parent zone have associated AAAA records, resolution via IPv6 is not possible.

Missing GLUE: If the name from an NS record for a zone is in-bailiwick, i.e., the name is within the zone or below, a parent zone must contain an IPv6 GLUE record, i.e., a parent must serve the corresponding AAAA record(s) as ADDITIONAL data when returning the NS record in the ANSWER section.

No AAAA record for in-bailiwick NS : If an NS record of a zone points to a name that is in-bailiwick but the name lacks AAAA record(s) in its zone, IPv6-only resolution will fail even if the parent provides GLUE, when the recursive server validates the delegation path. One such example is Unbound with the setting harden-glue: yes–the default.

Zone of out-of-bailiwick NS es not resolving: If an NS record of a zone is out-of-bailiwick, the corresponding zone must be IPv6-resolvable as well. It is insufficient if the name pointed to by the NS record has an associated AAAA record.

Parent zone not IPv6-resolvable: For a zone to be resolvable via IPv6 the parent zones up to the root zone must be IPv6-resolvable. Any non-IPv6-resolvable zone breaks the delegation chain for all its children.

The above misconfigurations are not mutually exclusive.


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
