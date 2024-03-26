---
title: "Discovery of Network Rate-Limit Policies Using Router Advertisements and DHCP"
abbrev: "Rate-Limit Policies Discovery"
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
        target: https://www.iana.org/assignments/icmpv6-parameters/
        date: false

     IANA-BOOTP:
        title: BOOTP Vendor Extensions and DHCP Options
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/bootp-dhcp-parameters/
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
properties are link MTU (RFC 4861) and PREFIX64 (RFC 8781). This document complements these tools and specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. For address family parity, a new DHCP option is also defined.

--- middle

# Introduction

## Context

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE) (see {{ac}}).
To facilitate data transfer via the provider network, it is assumed that appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
allowing successfully data exchange over these links. The required setup is referred to in this document as Attachment Circuits (ACs),
while the underlying link is referred to as "bearers".

The bearer can be a physical or logical link that connects a customer device to a provider network. A bearer can be a wireless or wired link.

~~~~aasvg
     .-------.                .--------------------.         .-------.
     |       +------+         |                    +---AC----+       |
     | UE#1  |      |         |                    +---AC----+ CE#2  |
     '-------'      +---AC----+                    |         '-------'
                              |     Network        |
     .-------.      .---AC----+                    |
     |       |      |         |                    |         .-------.
     | CE#1  +------'         |                    +---AC----+ CE#3  |
     '-------'                |                    |         '----+--'
        /|\                   '-----------+--------'              |
       O O O                              |                       |
       Hosts                              '-----------AC----------'
~~~~
{: #ac title="Sample Attachment Circuits " artwork-align="center"}

Customer terminating points are provided with a set of information (e.g., IP address/prefix) to successfully be
able to send and receive traffic over an attachment circuit. A comprehensive list of provisioning parameters that are available on
the PE-side of an attachment circuit is documented in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

The required set of parameters is a function of the service offering. For example, a very limited set of parameters is required for mass-market
service offering while a more elaborated set is required for Enterprise services (e.g., Layer 2 or Layer 3 VPN services). This document
**leverages access control, authorization, and authentication mechanisms that are already in place for the delivery of services over these attachment circuits**.

## Networks Are Already Sharing Network Properties with Hosts

To optimally deliver connectivity services, networks also advertize a set of information to connected hosts such as:

Link Maximum Transmission Unit (MTU):
: For example, the 3GPP {{TS-23.501}} specifies that "the link MTU size for IPv4 is sent to the UE by including it in the PCO (see TS 24.501). The link MTU size for IPv6 is sent to the UE by including it in the IPv6 Router Advertisement message (see RFC 4861)".
: {{Section 2.10 of ?RFC7066}} indicates that a cellular host should honor the MTU option in the Router Advertisement ({{Section 4.6.4 of !RFC4861}}) given that the 3GPP system
architecture uses extensive tunneling in its packet core network below the 3GPP link, and this may lead to packet fragmentation issues.

Prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) {{?RFC8781}}:
: This option is useful to enable local DNSSEC validation, support networks with no DNS64, support IPv4 address literals on an IPv6-only host, etc.

Encrypted DNS option {{?RFC9463}}:
: This option is used to discover encrypted DNS resolvers of a local network.

## What's In?

{{?I-D.rwbr-tsvwg-signaling-use-cases}} discusses some use cases where it is beneficial to share policies with the hosts. **Given that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}**, this document specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. For address family parity, a DHCP option {{!RFC2132}} is also defined for IPv4.

These options are called: Network Rate-Limit Policy (NRLP).

This document uses the host/network metadata specified in {{Section 5.1 of !I-D.rwbr-sconepro-flow-metadata}}.

In order to ensure consistent design for both IPv4 and IPv6 attachment circuits, {{sec-blob}} groups the set of NRLP parameters that are returned independent of the address family. This blob can be leveraged in networks where DHCP is not used and ease the mapping with specific protocols used in these networks. For example, a PCO NRLP IE can be defined in 3GPP.

Whether host-to-network, network-to-host, or both policies are returning an NRLP is deployment specific. All these combinations are supported in this document.

Also, the design supports returning one more NRLP instances for a given traffic direction.

## Design Motivation & Rationale

The main motivations for the use of ND for such a discovery are listed in {{Section 3 of ?RFC8781}}:

* Fate sharing
* Atomic configuration
* Updatability: change the policy at any time
* Deployability

The solution specified in the document is designed to **ease integration with network managment tools** that are used to manage and expose policies. It does so by leveraging the policy structure defined in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

The solution defined in this document:

* **Does not require any data plane change**.
* **Supports cascaded environments** where multiple levels to enforce rate limiting polices is required (e.g., WAN and LAN).
* **Supports signaling policies bound to one or both traffic directions**.

Compared to a proxy or an encapsulation-based proposal (e.g., {{?I-D.ihlar-masque-sconepro-mediabitrate}}), the solution defined in this document:

* **Does not impact the MTU tweaking**: No packet overhead is required.

<!--
* **Does not suffer from side effects of multi-layer encryption schemes** on the packet processing and overall performance of involved network nodes.
* **Does not suffer from nested congestion control**.
-->

* **Does not suffer from the complications of IP address sharing {{?RFC6269}}**. Such issues are likely to be experienced for proxy-based solutions that multiplex internal connections using one or more external IP addresses.
* **Requires a minor change to the network**: For IPv6, upgrade PE nodes to support a new ND option. Note that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}. For IPv4, configure DHCP server to include new DHCP option.

## What's Out?

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates an attachment circuit.

Likewise, the document does not make any assumption about the services or applications that are delivered over an attachment circuit. Whether one or multiple services
are bound to the same attachment circuit is deployment specific.

This document does not specify how a receiving host uses the discovered policy. Readers should refer, e.g., to {{?I-D.rwbr-tsvwg-signaling-use-cases}} for some examples.

## Sample Deployment Cases

Some deployment use cases for NRLP are provided below:

* A network may advertize an NRLP when it is overloaded, including when it is under attack. The rate limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events.

* Discovery of rate limit policy applied on attachment circuits (peering links, CE-PE links, etc.).

* A user may configure policies on the CPE such as securing some resources to a specific internal host used for gaming or video streaming. The CPE can use the NRLP option to share these rate limit policies to connected hosts to adjust their forwarding behavior.

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

# NRLP Blob {#sec-blob}

This section defines the set of attributes that are included in an NRLP blob:

D:
: 1-bit flag which indicates the direction on which to apply the enclosed policy.
: When set to "1", this flag indicates that this policy is for
  network-to-host direction.
: When set to "0", this flag indicates that this policy is for
  host-to-network direction.

E:
: When set to "1", this flag indicates the presence of Excess Information Rate (EIR).
: When set to "0", this flag indicates that EIR is not present.

P:
: When set to "1", this flag indicates the presence of Peak Information Rate (PIR).
: When set to "0", this flag indicates that PIR is not present.

U:
: Unassigned bit.

Scope:
: 4-bit field which specifies whether the policy is per host, per subscriber, etc.
: The following values are supported:

  + "0": Subscriber
  + "1": Host
  + 2-15: Unassigned values.

TC:
: 8-bit field which specifies a traffic category to which this policy applies.
: The following values are supported:

  + "0": All traffic. This is the default value.
  + "1": Streaming
  + "2":     Realtime
  + "3": Bulk trafic
  + 4-255: Unassigned values

Committed Information Rate (CIR) (Mbps):
: Specifies the maximum number of bits that a network can receive or
  send during one second over an attachment circuit for a
  traffic category.
: If set to 0, this indicates to the host that an alternate path (if any) should be preferred over this one.
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.
: This parameter is mandatory.

Committed Burst Size (CBS) (bytes):
: Specifies the maximum burst size that can be transmitted at CIR.
: MUST be greated than zero.
: This parameter is mandatory.

Excess Information Rate (EIR) (Mbps):
: MUST be present only if the E flag is set to '1'.
: Specifies the maximum number of bits that a network can receive or
  send during one second over an attachment circuit for a
  traffic category that is out of profile.
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.
: This parameter is optional.

Excess Burst Size (EBS) (bytes):
: MUST be present only if EIR is also present.
: Indicates that maximum excess burst size that is allowed while not complying with the CIR.
: MUST be greater than zero, if present.
: This parameter is optional.

Peak Information Rate (PIR) (Mbps):
: MUST be present only if P flag is set to '1'.
: Traffic that exceeds the CIR and the CBS is metered to the PIR.
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.
: This parameter is optional.

Peak Burst Size (PBS) (bytes):
: MUST be present only if PIR is also present.
: Specifies the maximum burst size that can be transmitted at PIR.
: MUST be greater than zero, if present.

The reader should refer to {{?RFC2697}}, {{?RFC2698}}, and {{?RFC4115}} for examples
of how various combinations of CIR/CBS/EIR/EBS/PIR/PBS are used for policing. Typically:

* A Single-Rate, Three-Color Marker {{?RFC2697}} uses CIR, CBS, and EBS.
* A Dual-Rate, Three-Color Marker {{?RFC2698}} uses CIR, CBS, PIR, and PBS.

# IPv6 RA NRLP Option

## Option Format

The format of the IPv6 RA NRLP option, with only mandatory fields included, is illustrated in {{opt-m-format}}.

~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Length    |D|E|P|U| Scope |      TC       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                Committed Information Rate (CIR)               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  Committed Burst Size (CBS)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #opt-m-format title="NRLP Option Format with Mandatory Fields" artwork-align="center"}

The format of the IPv6 RA NRLP option, with optional fields included, is illustrated in {{opt-m-format}}.

~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Length    |D|E|P|U| Scope |      TC       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                Committed Information Rate (CIR)               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  Committed Burst Size (CBS)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Excess Information Rate (EIR) (Optional)        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  Excess Burst Size (EBS) (Optional)           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Peak Information Rate (PIR) (Optional)          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                  Peak Burst Size (PBS) (Optional)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #opt-format title="NRLP Option Format with Optional Fields" artwork-align="center"}

The fields of the option shown in {{opt-format}} are as follows:

Type:
: 8-bit identifier of the NRLP option as assigned by IANA (TBD1).

Length:
: 8-bit unsigned integer.  The length of the option (including
  the Type and Length fields) is in units of 8 octets.

D:
: See {{sec-blob}}.

E:
: See {{sec-blob}}.

P:
: See {{sec-blob}}.

U:
: Unassigned bit.

Scope:
: See {{sec-blob}}.

TC:
: See {{sec-blob}}.

Committed Information Rate (CIR) (Mbps):
: See {{sec-blob}}.

Committed Burst Size (CBS) (bytes):
: See {{sec-blob}}.

Excess Information Rate (EIR) (Mbps):
: See {{sec-blob}}. This is an optional field.

Excess Burst Size (EBS) (bytes):
: See {{sec-blob}}. This is an optional field.

Peak Information Rate (PIR) (Mbps):
: See {{sec-blob}}. This is an optional field.

Peak Burst Size (PBS) (bytes):
: See {{sec-blob}}. This is an optional field.

## IPv6 Host Behavior

The procedure for rate-limit configuration is the same as it is with any
other Neighbor Discovery option {{!RFC4861}}.

The host MUST be prepared to receive multiple NRLP options
in RAs; each with distinct scope and/or application group.

If the host receives multiple NRLP options with overlapping scope/TC, the host MUST silently discard all these options.

If the receiving host is a CE (e.g., mobile CE or mobile handset with tethering), the following behavior applies:

* If an RA NRLP is advertised from the network, and absent local rate limit policies, the
device should send RAs to the downstream attached LAN devices with the same NRLP values received from the network.

* If local rate-limit policies are provided to the device, the device may change the scope or values received from the network
to accommodate these policies. The device may decide to not relay received RAs to internal nodes if local policies were
already advertized using RAs and those policies are consistent with the network policies.

Applications running over a host can learn the bitrates associated with a network attachment by invoking a dedicated API. The exact details of the API is OS-specific and, thus, out of scope of this document.

# DHCP NRLP Option

> Note that DHCP can only signal a rate policy change when the
  client first joins the network or renews its lease, whereas IPv6 ND
  can update the rate policy at the network's discretion.

## Option Format

The format of the DHCP NRLP option is illustrated in {{dhc-format}}.

~~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| OPTION_V4_NRLP|     Length    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~      NRLP Instance Data #1    ~
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+   ---
.              ...              .    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ optional
~      NRLP Instance Data #n    ~    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+   ---
~~~~
{: #dhc-format title="NRLP DHCP Option Format" artwork-align="center"}

The fields of the option shown in  {{dhc-format}} are as follows:

Code:
: OPTION_V4_NRLP (TBD2).

Length:
: Indicates the length of the enclosed data in octets.

NRLP Instance Data:
: Includes a network rate-limit policy. The format of this field with only mandatory parameters is shown in {{nrlp-m-format}}.
: When several NRLPs are to be included, the "NRLP Instance Data" field is repeated.

~~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   NRLP Instance Data Length   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|D|E|P|U| Scope |      TC       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Information Rate   |
|              (CIR)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Burst Size (CBS)   |
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: #nrlp-m-format title="NRLP Instance Data Format with Mandatory Fields" artwork-align="center"}

The format of this field, with optional parameters included, is shown in {{nrlp-m-format}}.

~~~~
 0                   1
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   NRLP Instance Data Length   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|D|E|P|U| Scope |      TC       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Information Rate   |
|              (CIR)            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Committed Burst Size (CBS)   |
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
|  Excess Information Rate      |  |
|             (EIR)             |  O
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  P
|    Excess Burst Size (CBS)    |  T
|                               |  I
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  O
|    Peak Information Rate      |  N
|             (PIR)             |  A
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  L
|      Peak Burst Size (PBS)    |  |
|                               |  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ ---
~~~~
{: #nrlp-format title="NRLP Instance Data Format with Optional Fields Included" artwork-align="center"}

The fields shown in {{nrlp-format}} are as follows:

NRLP Instance Data Length:
: Length of all following data in octets. This field is set to '8' when only the nominal bitrate is provided for an NLRP instance.

D:
: See {{sec-blob}}.

E:
: See {{sec-blob}}.

P:
: See {{sec-blob}}.

U:
: Unassigned bit.

Scope:
: See {{sec-blob}}.

TC:
: See {{sec-blob}}.

Committed Information Rate (CIR) (Mbps):
: See {{sec-blob}}.

Committed Burst Size (CBS) (bytes):
: See {{sec-blob}}.

Excess Information Rate (EIR) (Mbps):
: See {{sec-blob}}. This is an optional field.

Excess Burst Size (EBS) (bytes):
: See {{sec-blob}}. This is an optional field.

Peak Information Rate (PIR) (Mbps):
: See {{sec-blob}}. This is an optional field.

Peak Burst Size (PBS) (bytes):
: See {{sec-blob}}. This is an optional field.

OPTION_V4_NRLP is a concatenation-requiring option. As such, the mechanism specified in {{!RFC3396}} MUST be used if OPTION_V4_NRLP exceeds the maximum DHCP option size of 255 octets.

## DHCPv4 Client Behavior

To discover a network rate-limit policy, the DHCP client includes OPTION_V4_NRLP in a Parameter Request List option {{!RFC2132}}.

The DHCP client MUST be prepared to receive multiple "NRLP Instance Data" field entries in the OPTION_V4_NRLP option; each instance is to be treated as a separate network rate-limit policy.

# Operational Considerations

NRLP senders should be configured with instructions about the type of network rate-limit policies to be shared with requesting hosts. These types can be provided using mechanisms such as {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

In contexts where the bitrate policies are known during the establishment of the underlying bearer (e.g., GBR PDU Sessions), sending NRLP signals over the attachment circuit may be redundant and should thus be disabled.

In contexts where the (average) bitrate policies provided during the establishment of a bearer cannot be refreshed to echo network-specific conditions (e.g., overload) using bearer-specific mechanisms, sending NRLP signals over the attachment circuit would allow control the load at the source.

When both bearer-specific policies and NRLP signals are communicated to a host, the NRLP signals takes precedence.

Rate-limit policies enforced at the network are assumed to be consistent with the local jurisdictions.

# Security Considerations

## ND

As discussed in {{?RFC8781}}, because RAs are required in all IPv6 configuration scenarios, RAs must already be secured, e.g., by deploying an RA-Guard {{?RFC6105}}. Providing all configuration in RAs reduces the attack surface to be targeted by malicious attackers trying to provide hosts with invalid configuration, as compared to distributing the configuration through multiple different mechanisms that need to be secured independently.

RAs are already used in mobile networks to advertize the link MTU. The same security considerartions for MTU discovery apply for the NRLP discover.

An attacker who has access to the RAs exchanged over an attachment circuit may:

*     Decrease the bitrate: This may lower the perceived QoS if the host aggressively lowers its transmission rate.
*     Increase the bitrate value: The attachment circuit will be overloaded, but still the rate-limit at the network will discard excess traffic.
*     Drop RAs: This is similar to the current operations, where no NRLP RA is shared.
*     Inject fake RAs: The implications are similar to the impacts of tweaking the values of a legitimate RA.

## DHCP

An attacker who has access to the DHCP exchanged over an attachment circuit may do a lot of harm (e.g., prevent access to the network).

The following mechanisms may be considered to mitigate spoofed or modified DHCP responses:

DHCPv6-Shield {{?RFC7610}}:
: The network access node (e.g., a border router, a CPE, an Access Point (AP)) discards DHCP response messages received from any local endpoint.

Source Address Validation Improvement (SAVI) solution for DHCP {{?RFC7513}}:
: The network access node filters packets with forged source IP addresses.

The above mechanisms would ensure that the endpoint receives the correct NRLP information, but these mechanisms cannot provide any information about the DHCP server or the entity hosting the DHCP server.

# IANA Considerations

## Neighbor Discovery Option

This document requests IANA to assign the following new IPv6 Neighbor Discovery Option
type in the "IPv6 Neighbor Discovery Option Formats" subregistry under the "Internet Control Message Protocol version 6 (ICMPv6)
Parameters" registry maintained at {{IANA-ND}}.

|Type|     Description|     Reference|
|TBD1|  NRLP Option|This-Document|
{: #iana-new-op title="Neighbor Discovery NRLP Option"}

## DHCP Option

This document requests IANA to assign the following new DHCP Option Code in the "BOOTP Vendor Extensions and DHCP Options" registry maintained at {{IANA-BOOTP}}.

|Tag|     Name|     Data Length|     Meaning|Reference|
|TBD2|OPTION_V4_NRLP|N|NRLP Option|This-Document|
{: #iana-new-dhcp title="DHCP NRLP Option"}

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
