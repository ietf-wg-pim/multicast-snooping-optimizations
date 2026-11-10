---
coding: utf-8

title: Proxy Query Messages
abbrev: Proxy Query Messages
docname: draft-karstens-pim-proxy-query-messages-00
date: 2026-11-02
updates:
  - 4541
  - 9776
  - 9777
category: std
submissionType: IETF

standalone: yes
pi: [toc, sortrefs, symrefs, comments]

ipr: trust200902
area: Routing
wg: pim
kw:
  - multicast
  - igmp
  - group
  - management
  - mld
  - listener
  - discovery
  - snooping

author:
  -
    ins: N. Karstens
    name: Nate Karstens
    org: Garmin International
    abbrev: Garmin
    email: nate.karstens@garmin.com

normative:
  BRIDGE:
    title: >
      IEEE Standard for Local and metropolitan area networks, Media Access Control (MAC) Bridges
    author:
      org: IEEE
    date: 2004
    seriesinfo:
      IEEE: Std 802.1D-2004
  RFC4541:
    -: SNOOP
    display: SNOOP
  RFC9776:
    -: IGMPv3
    display: IGMPv3
  RFC9777:
    -: MLDv2
    display: MLDv2

informative:
  RFC2710:
    -: MLDv1
    display: MLDv1
  RFC3590:

--- abstract

Some IGMP/MLD snooping switches send Queries, known as Proxy Queries, to solicit Membership Reports from hosts, for example after a Spanning Tree topology change. RFC 4541 describes how these Queries are identified and asks that they be excluded from Querier election, but as an Informational document it does not update the IGMP and MLD specifications, and its recommendation for MLD conflicts with them. This document specifies how multicast routers identify Proxy Queries and exclude them from Querier election: by the source address 0.0.0.0 in IGMP, and by a new P (Proxy Query) flag in MLD. This document updates RFCs 4541, 9776, and 9777.

--- middle

# Introduction {#introduction}

IGMP snooping is a feature included in many managed switches that provides optimal distribution of multicast traffic on the LAN. It works by having the switch, which typically operates at the data link layer, observe IGMP Membership Report messages to learn which link(s) are interested in a given multicast group and adjust an internal forwarding database according. The IPv6 equivalent, MLD snooping, behaves in a similar manner, observing Multicast Listener Report messages.

This operation blurs the functional boundaries between layers in the network stack. It also involves standards from two organizations: IEEE and IETF. Unfortunately, there is no authoritative specification for this operation. Instead, it was developed by vendors and later documented in {{!-SNOOP}}, an Informational RFC that describes the operation and records differences in vendor implementations.

{{!-SNOOP, Section 2.1.1}}, list item 4 describes a Proxy Query, a special-purpose Membership Query message with an IPv4 source address of `0.0.0.0`. A Proxy Query is a Membership Query message that must not be included in Querier election, but are otherwise handled as a normal Query.

{{!-SNOOP, Section 3}} describes the equivalent message in MLD and indicates that it should use the IPv6 unspecified address (`::`). However, several documents prohibit this:

* {{?-MLDv1, Section 3}} requires all MLD messages to be sent with an IPv6 link-local source address
* {{?RFC3590, Section 4}} requires MLD Query messages to be sent with a valid IPv6 link-local source address
* {{!-MLDv2, Section 5.1.14}} requires both that 1) MLD Query messages be sent with a valid IPv6 link-local source address and that 2) nodes discard Query messages without a valid IPv6 link-local source address

Because the recommendation to use the IPv6 unspecified address came from an Informational RFC, it does not overrule the precedent established in these authoritative documents. This raises the question of how Proxy Query messages should be implemented in MLD.

Similarly, because the concept of a Proxy Query message is only described in an Informational RFC, both the authority and applicability of this message are uncertain.

This document does the following to resolve these concerns:

1. Updates {{!-IGMPv3}} and {{!-MLDv2}} to specify operation of a Proxy Query
2. Formally removes the recommendation to use the IPv6 unspecified address for Proxy Query messages in MLD

Unlike {{!-SNOOP}}, this document can be considered authoritative because it affects {{!-IGMPv3}} and {{!-MLDv2}}, both IETF protocols.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

# IGMP Proxy Query {#igmp-proxy-query}

If an IPv4 multicast router receives an IGMP Membership Query with its IPv4 source address set to `0.0.0.0`, then it MUST NOT use that message in the querier election process described in {{!-IGMPv3, Section 6.6.2}}. If the IPv4 source address is not set to `0.0.0.0`, then the multicast router MUST use that message in the querier election process. All other handling for this message remains unchanged.

# MLD Proxy Query

Instead of working against precedent established by the documents listed in {{introduction}}, this document requests assignment of a new flag in the "IGMP/MLD Query Message Flags" registry: the P (Proxy Query) flag. To conserve flags and avoid duplicate functionality, this assignment is only made for MLD Query messages; the flag remains unassigned for IGMP Query messages.

If an IPv6 multicast router receives an MLD Multicast Listener Query with the P flag set to 1, then it MUST NOT use that message in the querier election process described in {{?-MLDv2, Section 7.6.2}}. If the P flag is set to 0, then the multicast router MUST use that message in the querier election process. All other handling for this message remains unchanged.

This document updates {{!-SNOOP, Section 3}} to formally withdraw the recommendation to use the IPv6 unspecified address for Proxy Query messages in MLD.

# Operational Considerations

The attributes that identify a Proxy Query were chosen such that no updates are needed for networks that do not use Proxy Query messages.

Operators deploying devices that send IGMP Proxy Query messages MUST ensure that any IPv4 multicast router they want to serve as Querier processes IGMP Membership Query messages as per {{igmp-proxy-query}} of this document, which is based on the recommendation in {{!-SNOOP, Section 2.1.1}}, list item 4. This ensures that the appropriate router will participate in Querier election and continue to send IGMP Membership Query messages as per {{!-IGMPv3, Section 6.6.2}}.

Operators deploying devices that send MLD Proxy Query messages onto a network with one or more IPv6 multicast routers that do not support these messages MUST ensure that any router they want to serve as Querier is given a lower IPv6 address than any device that sends Proxy Query messages. This ensures that the appropriate router will participate in Querier election and continue to send MLD Multicast Listener Query messages as per {{?-MLDv2, Section 7.6.2}}.

# Security Considerations

Proxy Queries are subject to the same validation as other IGMP Membership Queries {{!-IGMPv3}} and MLD Multicast Listener Queries {{!-MLDv2}}. Because routers exclude Proxy Queries from Querier election, a device cannot become the Querier by sending them; a device that wants to influence Querier election can already do so with ordinary Queries, as described in the security considerations of {{!-IGMPv3}} and {{!-MLDv2}}. Like any Query, a Proxy Query causes hosts to send reports, so a device that sends Proxy Queries at a high rate can increase report traffic on the link. In IGMP, Proxy Queries use the source address 0.0.0.0, so their sender cannot be identified from the source address.

# IANA Considerations

IANA should allocate a new flag in the IGMP/MLD Query Message Flags Registry with the following properties:

* Flags Bit: 1
* Short Name: P
* Description: "Proxy Query"
* Reference: this document

--- back
