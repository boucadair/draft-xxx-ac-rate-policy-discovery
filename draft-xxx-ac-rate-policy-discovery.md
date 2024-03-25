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
area: wit
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

     TS-23.501:
        title: "TS 23.501: System architecture for the 5G System (5GS)"
        date: 2024
        author:
        -
          org: 3GPP
        target: https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3144

--- abstract

Traffic exchanged over an attachment circuit may be subject to rate limit policies.
These policies may be intentional policies (e.g., enforced as part of the activation of the attachment circuit)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

Networks already support mechanisms to advertize a set of network properties to hosts using Neighbor Discovery options. Examples of such
properties are link MTU (RFC 4861) and PREFIX64 (RFC 8781). This document complements these tools and specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts.

--- middle

# Introduction

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE).
To facilitate data transfer via the provider network, it is assumed that the appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
allowing successfully data exchanged over these links. The required setup is referred to in this document as Attachment Circuits (ACs),
while the underlying link is referred to as "bearers".

Customer terminating points are also provided with a set of information (e.g., IP address/prefix) to successfully be
able to send and receive traffic over an attachment circuits. To optimally deliver connectivity services, networks
also advertize other information to connected hosts such as:

* Link Maximum Transmission Unit (MTU) {{!RFC4861}} to avoid fragmentation:
: For example, the 3GPP {{TS-23.501}} specifies that "the link MTU size for IPv4 is sent to the UE by including it in the PCO (see TS 24.501). The link MTU size for IPv6 is sent to the UE by including it in the IPv6 Router Advertisement message (see RFC 4861)".
: {{Section 2.10 of ?RFC7066}} indicates that a cellular host should honor the MTU option in the Router Advertisement ({{Section 4.6.4 of !RFC4861}}) given that the 3GPP system
architecture uses extensive tunneling in its packet core network below the 3GPP link, and this may lead to packet fragmentation issues.

* Prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) {{?RFC8781}}:
: This option is useful to enable local DNSSEC validation, support networks with no DNS64, support IPv4 address literals on an IPv6-only host, etc.

* Encrypted DNS option {{?RFC9463}} to discover encrypted DNS resolvers of a local network.

{{?I-D.rwbr-tsvwg-signaling-use-cases}} discusses some use cases where is beneficial to share policies to hosts. **Given that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}**, this document  specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. The motivation for the use of ND for such a discovery is listed in {{Section 3 of ?RFC8781}}:

* Fate sharing
* Atomic configuration
* Updatability: change the policy at any time.
* Deployability

The solution defined in this document:

* **Does not require any data plane change**.
* **Supports cascaded environments** where multiple levels to enforced rate limiting polices is required (e.g., WAN and LAN).

Compared to proxy or encapsulation proposals, the solution defined in this document:

* **Does not impact the MTU tweaking**.
* **Does not suffer from side effects of multi-layer encryption schemes** on the packet processing and overall performance of involved network nodes.
* **Does not suffer from nested congestion control**.
* **Requires a minor change to the network**: upgrade PE nodes to support a new ND option. Note that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}.

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates an attachment circuit.

Likewise, the document does not make assumption about the services or applications that are delivered over an attachment circuit. Whether one or multiple services
are bound to the same attachment circuit is deployment specific.

This document does not specify how a receiving host uses the discovered policy. Readers should refer, e.g., to {{?I-D.rwbr-tsvwg-signaling-use-cases}} for some examples.

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

As discussed in {{?RFC8781}}, because RAs are required in all IPv6 configuration scenarios, RAs must already be secured, e.g., by deploying an RA-Guard {{?RFC6105}}. Providing all configuration in RAs reduces the attack surface to be targeted by malicious attackers trying to provide hosts with invalid configuration, as compared to distributing the configuration through multiple different mechanisms that need to be secured independently.

XXXX



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
