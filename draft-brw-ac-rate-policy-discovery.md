---
title: "Discovery of Network Rate-Limit Policies in Router Advertisements"
abbrev: "Rate-Limit Policies in RAs"
category: std

docname: draft-brw-ac-rate-policy-discovery-latest
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
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    country: India
    email: kondtir@gmail.com
 -
    fullname: Dan Wing
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    country: United States of America
    email: danwing@gmail.com

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

Link Maximum Transmission Unit (MTU) {{!RFC4861}} to avoid fragmentation:
: For example, the 3GPP {{TS-23.501}} specifies that "the link MTU size for IPv4 is sent to the UE by including it in the PCO (see TS 24.501). The link MTU size for IPv6 is sent to the UE by including it in the IPv6 Router Advertisement message (see RFC 4861)".
: {{Section 2.10 of ?RFC7066}} indicates that a cellular host should honor the MTU option in the Router Advertisement ({{Section 4.6.4 of !RFC4861}}) given that the 3GPP system
architecture uses extensive tunneling in its packet core network below the 3GPP link, and this may lead to packet fragmentation issues.

Prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) {{?RFC8781}}:
: This option is useful to enable local DNSSEC validation, support networks with no DNS64, support IPv4 address literals on an IPv6-only host, etc.

Encrypted DNS option {{?RFC9463}}:
: This option is used to discover encrypted DNS resolvers of a local network.

{{?I-D.rwbr-tsvwg-signaling-use-cases}} discusses some use cases where it is beneficial to share policies to hosts. **Given that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}**, this document specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. This option is called: Network Rate-Limit Policy (NRLP).

The main motivations for the use of ND for such a discovery are listed in {{Section 3 of ?RFC8781}}:

* Fate sharing
* Atomic configuration
* Updatability: change the policy at any time.
* Deployability

The solution defined in this document:

* **Does not require any data plane change**.
* **Supports cascaded environments** where multiple levels to enforce rate limiting polices is required (e.g., WAN and LAN).

Compared to proxy or encapsulation proposals (e.g., {{?I-D.ihlar-masque-sconepro-mediabitrate}}), the solution defined in this document:

* **Does not impact the MTU tweaking**.
* **Does not suffer from side effects of multi-layer encryption schemes** on the packet processing and overall performance of involved network nodes.
* **Does not suffer from nested congestion control**.
* **Requires a minor change to the network**: upgrade PE nodes to support a new ND option. Note that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}.

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates an attachment circuit.

Likewise, the document does not make any assumption about the services or applications that are delivered over an attachment circuit. Whether one or multiple services
are bound to the same attachment circuit is deployment specific.

This document does not specify how a receiving host uses the discovered policy. Readers should refer, e.g., to {{?I-D.rwbr-tsvwg-signaling-use-cases}} for some examples. Some deployment use cases for NRLP are provided below:

* A network may advertize a NRLP when it is overloaded, including when it is under attack. The rate limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events.

* Discovery of rate limit applied on attachment circuits (peering links, CE-PE links, etc.).

* A user may configure policies on the CPE such as securing some resources to a specific internal host used for gaming or video streaming. The CPE can use the RA NRLP to share these rate limit policies to each these connected hosts to adjust their forwarding behavior.

This document uses the host/network metadata specified in {{Section 5.1 of !I-D.rwbr-sconepro-flow-metadata}}.

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

# IPv6 RA NRLP Option

## Option Format

The format of the IPv6 RA NRLP option is illustrated in {{opt-format}}.

~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Length    |D|   Scope     |      TC       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         nominal bitrate                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           burst bitrate (optional)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             duration   (optional)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #opt-format title="NRLP Option Format" artwork-align="center"}

The fields of the option shown in {{opt-format}} are as follows:

Type:
: 8-bit identifier of the NRLP option as assigned by IANA (TBD).

Length:
: 8-bit unsigned integer.  The length of the option (including
  the Type and Length fields) is in units of 8 octets.
: If the "Length" is set to "8", this indicates that only the
nominal bitrate is provided.

D:
: 1-bit flag which indicates the direction on which to apply the enclosed polciy.
: When set to "1", this flag indicates that this policy is for
  network-to-host direction.
: When set to "0", this flag indicates that this policy is for
  host-to-network direction.

Scope:
: 7-bit field which specifies whether the policy is per host, per subscriber, etc.
: The following values are supported:

  + "0": subscriber
  + "1": host
  + "2": application group

TC:
: 8-bit field which specifies a traffic category to which this policy applies.
: The following values are supported:

  + "0": all traffic
  + "1":	Browsing
  + "2": Streaming
  + "3":	Realtime
  + "4": Bulk
  + "5": background trafic

nominal bitrate (Mbps):
: Specifies the maximum number of bits that a network can receive or
  send during one second over an attachment circuit for a
  traffic category.
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.

burst bitrate (Mbps):
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}. This field is optional.

burst duration:
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}. This field MUST be present
only if a burst bitrate is present.

> Consider using {{?RFC4115}} (CIR/EIR).

## IPv6 Host Behavior

The procedure for rate-limit configuration is the same as it is with any
other Neighbor Discovery option {{!RFC4861}}.  In addition, the host
XXXX.

The host MUST be prepared to receive multiple NRLP options
in RAs; each with distinct scope and/or application group.

If the receiving host is a CE (e.g., mobile CE or mobile handset with tethering), the following behavior applies:

* If an RA NRLP is advertised from the network, and absent local rate limit policies, the
device should send RAs to the downstream attached LAN devices with the same NRLP values received from the network.

* If local rate-limit policies are provided to the device, the device may change the scope or values received from the network
to accommodate these policies. The device may decide to not relay received RAs to internal nodes if local policies were
already advertized using RAs and those policies are consistent with the network policies.

Application running over a host can learn the bitrates associated with a network attachment by invoking a dedicated API. The exact details of the API is OS-specific and, thus, out of scope of this document.

# Security Considerations

As discussed in {{?RFC8781}}, because RAs are required in all IPv6 configuration scenarios, RAs must already be secured, e.g., by deploying an RA-Guard {{?RFC6105}}. Providing all configuration in RAs reduces the attack surface to be targeted by malicious attackers trying to provide hosts with invalid configuration, as compared to distributing the configuration through multiple different mechanisms that need to be secured independently.

RAs are already used in mobile networks to advertize the link MTU. The same security considerartions for MTU discovery apply for the NRLP discover.

An attacker who has access to the RAs exchange over an attachment circuit may:

*	Decrease the bitrate: This may lower the perceived QoS if the host aggressively lowers its transmission rate.
*	Increase the bitrate value: The attachment circuit will be overloaded, but still the rate-limit at the network will discard excess traffic.
*	Drop RAs: This is similar to the current operations, where no NRLP RA is shared.
*	Inject fake RAs: The implications are similar to the impacts of tweaking the values of a legitimate RA.

# IANA Considerations

This document requests IANA to assign the following new IPv6 Neighbor Discovery Option
type in the "IPv6 Neighbor Discovery Option Formats" subregistry under the "Internet Control Message Protocol version 6 (ICMPv6)
Parameters" registry maintained at {{IANA-ND}}.

|Type|	Description|	Reference|
|TBD|  NRLP Option|This-Document|
{: #iana-new-op title="Neighbor Discovery NRLP Option"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
