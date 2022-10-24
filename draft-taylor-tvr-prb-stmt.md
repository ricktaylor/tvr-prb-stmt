---
title: "Time Variant Routing Problem Statement"
abbrev: "tvr-prb-stmt"
category: info

docname: draft-taylor-tvr-prb-stmt-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Time Variant Routing"
keyword:
 - time variant routing
 - time sensitive routing
 - TVR

venue:
  group: "Time Variant Routing"
  type: "Working Group"
  mail: "tvr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/tvr/"
  github: "ricktaylor/tvr-prb-stmt"
  latest: "https://ricktaylor.github.io/tvr-prb-stmt/draft-taylor-tvr-prb-stmt.html"

author:
 -
    fullname: Rick Taylor
    organization: Ori Industries
    email: rick.taylor@ori.co

normative:

informative:

--- abstract

Existing Routing Protocols expect to maintain contemporaneous, end-to-end connected paths across a network. Changes to that connectivity, such as the loss of an adjacent peer, are considered to be exceptional circumstances that must be corrected prior to the resumption of data transmission. Corrections may include attempting to re-establish lost adjacencies and recalculating or rediscovering a functional topology.

However, there are a growing number of use-cases where changes to the routing topology are an expected part of network operations. In these cases the pre-planned loss and restoration of an adjacency, or formation of an alternate adjacency, should be seen as a non-disruptive event.

This document attempts to describe the problems perceived with existing routing protocols and act as a "Problem Statement" to be addressed by the proposed Time-Variant Routing (TVR) working group.

Readers are recommended to read this document alongside the [TVR (Time-Variant Routing) Use Cases](https://datatracker.ietf.org/doc/draft-birrane-tvr-use-cases/).

--- middle

# Introduction

Existing Routing Protocols expect to maintain contemporaneous, end-to-end connected paths across a network. Changes to that connectivity, such as the loss of an adjacent peer, are considered to be exceptional circumstances that must be corrected prior to the resumption of data transmission. Corrections may include attempting to re-establish lost adjacencies and recalculating or rediscovering a functional topology.

However, there are a growing number of use-cases where changes to the routing topology are an expected part of network operations. In these cases the pre-planned loss and restoration of an adjacency, or formation of an alternate adjacency, should be seen as a non-disruptive event.

The expected loss (and planned resumption) of links can occur for a variety of reasons. In networks with mobile nodes, such as unmanned aerial vehicles and some orbiting spacecraft constellations, links are lost and re-established as a function of the mobility of the platforms. In networks without reliable access to power, such as networks harvesting energy from wind and solar, link activity might be restricted to certain times of day. Similarly in networks prioritizing green computing and energy efficiency over data rate, network traffic might be planned around energy costs or expected user data volumes.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Disclaimer

This document has been written rapidly, and a draft published prior to any serious peer-review; it therefore may be factually incorrect in its assumptions.  However, the author feels it is important to begin this conversation in order to discover if there is merit in examining the perceived issues detailed.

# Behaviour of existing Routing Protocols

Since the dawn of the Internet, existing routing protocols have been developed to ensure that end-to-end paths can be established for the transmission of datagrams.  As the Internet began on academic and research networks, the routing protocol designs have developed based on assumptions about the characteristics of these early networks, namely:

* The individual nodes in a network do not move around.
* The individual nodes are expected to be permanently available once they join the network.

Issues with the first assumption have long been known, and have been the subject of much research under the umbrella term Mobile Ad-hoc Networks (MANET).  The IETF has had an active Working Group attempting to standardise routing protocols specifically designed for that use-case for many years, however the second assumption has been largely ignored, and the result of this has been two-fold:

* A number of routing protocols, such as BGP {{?RFC4271}} and OSPF {{?RFC2328}}, have been standardised for use in extremely stable networks, where the loss of reachability to a node in the network is considered an exceptional circumstance that requires some kind of recovery mechanism, sometimes requiring some kind of global recalculation of the topology.

* A number of specialised MANET routing protocols, such as OLSR {{?RFC7181}} and AODV {{?RFC3561}}, have been standardised for use in high-churn environments, where nodes move and disappear from the network in unpredictable ways.  These protocols have novel approaches to mitigate against the disruption caused by these rapid unscheduled changes to the network topology.

However, the author believes there is a third issue with the underlying assumptions that have driven routing protocol design to date:  There are use-cases where one or more nodes in the network may become unreachable, or relocate within the current topology, at a time known in advance of it happening.  The belief is that these use-cases are not adequately satisfied by the current suite of routing protocols.

## Connectivity Loss

In both stable and MANET routing protocols, the loss of connectivity to one or more nodes is seen by their peers as an erroneous condition, and this has driven the focus of research into increasingly sophisticated connectivity monitoring techniques, ranging from simple timer-based "Hello" mechanisms, to rich interaction with the link-layer, for example DLEP {{?RFC8175}}.  In all cases these methods are designed to enable a router to promptly detect an adjacency failure, completely ignoring the use-cases where the timing of the failure is well-known in advance.

A second effect of treating peer-connectivity failures as exceptions to the correct operation of a network is that mitigation for the loss of the peer only begins once the adjacency failure has occurred.  If the loss of connectivity is predictable, then the (possibly expensive) route recalculations can have occurred well in advance of the loss happening, and executed at the exact moment they are needed, reducing disruption to the network as a whole.

## Connectivity Gain

In the use-cases where loss of connectivity with a peer is a predicted event, then often the start or resumption of connectivity is also known in advance of it happening.  As with loss of connectivity, existing protocols fall back to one or all of a set of common mechanisms:

* Timer-based polling of a previous adjacency, in case it came back.
* Reliance on some link-layer notification mechanism.
* Multicast-based discovery of unexpected new peers.

None of these mechanisms is particularly timely, as the expected arrival or return of a peer is seen as an unexpected event.  When connectivity with a potential peer is (re)established, generally a full re-introduction to the routing topology is conducted, which can be an expensive and potentially disruptive process.

# Problems to Solve

What is not being proposed is the development of a new alternative routing protocol to address the particular use case of expected failure, for a number of reasons:

1. Designing routing protocols is hard, and civilisation has many good ones to chose from already.

2. Even if some adjacency failures are predictable, others are not.  Any new routing protocol must still handle the case of unexpected failure.

Therefore the suggestion is to augment existing protocols with the capability to react to expected adjacency failure.

The author believes that the work on Time Variant Routing should be focussed on solving the following problems.

## Adjacency Schedules

In order to react to the expected failure and resumption of connectivity to a peer, relevant nodes within the domain of a single routing protocol instance must be made aware of the time and nature of the expected change in connectivity.  This information is referred to here as the Adjacency Schedule.

Although few protocols share an on-the-wire binary representation of data, there are common constructs, such as an IP subnet, that all protocols share.  We suggest that the same applies to an Adjacency Schedule.  It should be perfectly possible to define in some generic manner that shape and content of the data that describes a schedule, and then map that content to the relevant wire format of existing routing protocols.  A future IETF Time Variant Routing Working Group should define a common data model for an Adjacency Schedule, including the rules for correctly creating, updating and deleting the content.

## Distributing Adjacency Schedules

How an Adjacency Schedule is distributed amongst the peers of an individual routing protocol instance is dependant on many factors individual to each routing protocol, and a future IETF Time Variant Routing Working Group must work with the relevant subject matter experts, preferably a relevant IETF Working Group, in order to standardise the optimal distribution of an Adjacency Schedule to the correct peers for each routing protocol.

## Executing Adjacency Schedules

Of course, just knowing about an Adjacency Schedule is insufficient, changes in behaviour of routing protocols must be standardised such that advantage can be taken of having prior knowledge of expected changes in topology.  To achieve this a future IETF Time Variant Routing Working Group should determine the set of actions that can be scheduled, and then work with the relevant subject matter experts, preferably a relevant IETF Working Group, in order to standardise the execution of those actions, in the scope of each routing protocol.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
