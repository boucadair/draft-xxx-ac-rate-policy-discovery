---
title: "Discovery of Network Rate-Limit Policies in Router Advertisements"
abbrev: "Rate-Limit Policies in RAs"
category: std

docname: draft-xxx-ac-rate-policy-discovery-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - collaborative networking
 - adaptive application

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    email: mohamed.boucadair@orange.com

normative:

informative:
     IANA-ND:
        title: IPv6 Neighbor Discovery Option Formats
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/icmpv6-parameters/icmpv6-parameters.xhtml
        date: false

--- abstract

Traffic exchanged over an attachment circuit may be subject to rate limit policies.
These policies may be intentional policies (e.g., enforced as part of the activation of the attachment circuit)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

Networks already support mechanisms to advertize a set of network properties to hosts using Neighbor Discovery options. Examples of such
properties are link MTU (RFC 4861) and PREFIX64 (RFC 8781). This document complements these tools and specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts.

--- middle

# Introduction

TODO Introduction

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the terms defined in {{Section 2 of ?I-D.ietf-opsawg-ntw-attachment-circuit}}.

Also, this document makes use fo the following terms:

Reactive policy:
:  Treatment given to a flow when an exceptional event
   occurs, such as diminished throughput to the host caused by radio
   interference or weak radio signal, congestion on the network
   caused by other users or other applications on the same host.

Intentional policy:
:  Configured bandwidth, pps, or similar throughput
   constraints applied to a flow, application, host, or subscriber.

# Generalized Use Case

~~~~
          +--rw service
             +--rw mtu?                      uint32
             +--rw svc-pe-to-ce-bandwidth {vpn-common:inbound-bw}?
             |  +--rw bandwidth* [bw-type]
             |     +--rw bw-type      identityref
             |     +--rw (type)?
             |        +--:(per-cos)
             |        |  +--rw cos* [cos-id]
             |        |     +--rw cos-id    uint8
             |        |     +--rw cir?      uint64
             |        |     +--rw cbs?      uint64
             |        |     +--rw eir?      uint64
             |        |     +--rw ebs?      uint64
             |        |     +--rw pir?      uint64
             |        |     +--rw pbs?      uint64
             |        +--:(other)
             |           +--rw cir?   uint64
             |           +--rw cbs?   uint64
             |           +--rw eir?   uint64
             |           +--rw ebs?   uint64
             |           +--rw pir?   uint64
             |           +--rw pbs?   uint64
             +--rw svc-ce-to-pe-bandwidth {vpn-common:outbound-bw}?
             |  +--rw bandwidth* [bw-type]
             |     +--rw bw-type      identityref
             |     +--rw (type)?
             |        +--:(per-cos)
             |        |  +--rw cos* [cos-id]
             |        |     +--rw cos-id    uint8
             |        |     +--rw cir?      uint64
             |        |     +--rw cbs?      uint64
             |        |     +--rw eir?      uint64
             |        |     +--rw ebs?      uint64
             |        |     +--rw pir?      uint64
             |        |     +--rw pbs?      uint64
             |        +--:(other)
             |           +--rw cir?   uint64
             |           +--rw cbs?   uint64
             |           +--rw eir?   uint64
             |           +--rw ebs?   uint64
             |           +--rw pir?   uint64
             |           +--rw pbs?   uint64
~~~~

# IPv6 RA Encrypted DNS Option

## Option Format

## IPv6 Host Behavior

The procedure for rate-limit configuration is the same as it is with any
other Neighbor Discovery option {{!RFC4861}}.  In addition, the host
XXXX.

The host MUST be prepared to receive multiple xxx options
in RAs; each with distinct scope and/or application group.

# Security Considerations

TODO Security


# IANA Considerations

This document requests IANA to assign the following new IPv6 Neighbor Discovery Option
type in the "IPv6 Neighbor Discovery Option Formats" subregistry under the "Internet Control Message Protocol version 6 (ICMPv6)
Parameters" registry maintained at {{IANA-ND}}.

|Type|	Description|	Reference|
|TBD|  XXX Option|This-Document|
{: #iana-new-op title="Neighbor Discovery xxxx Option"}


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
