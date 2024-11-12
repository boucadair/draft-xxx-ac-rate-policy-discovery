---
title: "Throughput Advice Object for SCONE"
abbrev: "SCONE Blob"
category: std

docname: draft-brw-scone-throughput-advice-blob-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: wit
workgroup: scone
keyword:
 - collaborative networking
 - adaptive application

author:
 -
    fullname: Mohamed Boucadair
    organization: Orange
    email: mohamed.boucadair@orange.com
 -
    fullname: Dan Wing
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    country: United States of America
    email: danwing@gmail.com
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    country: India
    email: kondtir@gmail.com
 -
    fullname: Sridharan Rajagopalan
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    country: United States of America
    email: "sridharan.girish@gmail.com"
 -
    ins: L. Contreras
    name: Luis M. Contreras
    org: Telefonica
    country: Spain
    email: luismiguel.contrerasmurillo@telefonica.com

normative:

informative:
     TS-23.503:
        title: "TS 23.503: Policy and charging control framework for the 5G System (5GS)"
        date: 2024
        author:
        -
          org: 3GPP
        target: https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3334

     TS-29.522:
        title: "TS 29.522: 5G System; Network Exposure Function Northbound APIs"
        date: 2024
        author:
        -
          org: 3GPP
        target: https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3437

     app-measurement:
        title: "Bandwidth measurement for QUIC"
        date: 2024
        author:
        -
          fullname: Zafer Gurel
        -
          fullname: Ali C. Begen
        target: https://datatracker.ietf.org/doc/slides-119-moq-bandwidth-measurement-for-quic/

     TS-24.008:
        title: "Technical Specification Group Core Network and Terminals; Mobile radio interface Layer 3 specification; Core network protocols; Stage 3 (Release 18)"
        date: 2024
        author:
        -
          org: 3GPP
        target: https://www.3gpp.org/DynaReport/24008.htm

     BEREC:
        title: "All you need to know about Net Neutrality rules in the EU"
        date: false
        author:
        -
          org: BEREC
        target: https://www.berec.europa.eu/en/all-you-need-to-know-about-net-neutrality-rules-in-the-eu-0

     FCC:
        title: "FCC Restores Net Neutrality"
        date: false
        author:
        -
          org: FCC
        target: https://www.fcc.gov/document/fcc-restores-net-neutrality-0

     TR-470:
        title: "5G Wireless Wireline Convergence Architecture - Issue 2"
        date: false
        author:
        -
          org: BBF
        target: https://www.broadband-forum.org/pdfs/tr-470-2-0-0.pdf

--- abstract

Traffic exchanged over a network attachment may be subject to rate-limit policies.
These policies may be intentional policies (e.g., enforced as part of the activation
of the network attachment and typically agreed upon service subscription)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

This document specifies a generic object that can be be used by any mechanim for hosts
to dynamically discover Network Rate-Limit Policies (NRLPs). This information is then
passed to applicaitons that might adjust their behaviors accordingly.

--- middle

# Introduction

## Context

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE).
To facilitate data transfer via the provider network, it is assumed that appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
successfully allowing data exchange over these links. The required setup is referred to in this document as network attachments,
while the underlying link is referred to as "bearers".

The bearer can be a physical or logical link that connects a customer device to a provider network. A bearer can be a wireless or wired link. The same or multiple bearer technologies can be used to establish the bearer (e.g., WLAN, cellular) to graft customer terminating points to a network.

> Network attachment is also known as "Attachment Circuit (AC)" which is an established concept in the industry and also in the IETF ({{?RFC4026}}, {{?RFC4664}}, {{?RFC4364}}, etc.).

{{ac}} shows an example of a network that connects CEs and hosts (UE, for example).These CEs are servicing
other (internal) hosts. The identification of these hosts is hidden from the network. The policies enforced at the network
for an AC are per-subscriber, not per-host. Typically, if a CE is provided with a /56 IPv6 prefix, policies are enforced
on that /56 not the individual /64s that will be used by internal hosts. A customer terminating point may be serviced with one (e.g., UE#1, CE#1, and CE#3) or multiple ACs (e.g., CE#2).

~~~~aasvg
                                                        Hosts
                                                        O O O
                                                         \|/
.------.                .--------------------.         .------.
|      +------+         |                    +---AC----+      |
| UE#1 |      |         |                    +---AC----+ CE#2 |
'------'      +---AC----+                    |         '------'
                        |     Network        |
.------.      .---AC----+                    |
|      |      |         |                    |         .------.
| CE#1 +------'         |                    +---AC----+ CE#3 |
'------'                |                    |         '------'
   /|\                  '--------------------'            /|\
  O O O                                                  O O O
  Hosts                                                  Hosts
~~~~
{: #ac title="Sample Network Attachments" artwork-align="center"}

Customer terminating points are provided with a set of information (e.g., IP address/prefix) to successfully be
able to send and receive traffic over an AC. A comprehensive list of provisioning parameters that are available on
the PE-side of an AC is documented in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

The required set of parameters is a function of the service offering. For example, a very limited set of parameters is required for mass-market
service offering while a more elaborated set is required for Enterprise services (e.g., Layer 2 VPN {{?RFC9291}} or Layer 3 VPN {{?RFC9182}}). This document
**leverages access control, authorization, and authentication mechanisms that are already in place for the delivery of services over these ACs**.

In order to ensure consistent design for both IPv4 and IPv6 ACs, {{sec-blob}} groups the set of NRLP parameters that are returned independent of the address family. This blob can be leveraged in networks where ND/DHCP/PvD are not used and ease the mapping with specific protocols used in these networks. For example, ***a Protocol Configuration Option (PCO) {{TS-24.008}} NRLP Information Element can be defined in 3GPP***.

Whether host-to-network, network-to-host, or both policies are returned in an NRLP is deployment specific. All these combinations are supported in this document.

Also, the design supports returning one more NRLP instances for a given traffic direction.

## What's Out?

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates an AC.

Likewise, the document does not make any assumption about the services or applications that are delivered over an AC. Whether one or multiple services
are bound to the same AC is deployment specific.

Applications will have access to all these NRLPs and will, thus, adjust their behavior as a function of scope and traffic category indicated in a policy (all traffic, streaming, etc.). An application that couples multiple flow types will adjust each flow type to be consistent with the specific policy for the relevant traffic category. Likewise, a host with multiple ACs may use the discovered NRLPs AC to decide how to distribute its flows over these ACs (prefer an AC to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policy.


## Sample Deployment Cases

Some deployment use cases for NRLP are provided below:

* A network may advertize an NRLP when it is overloaded, including when it is under attack. The rate-limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events (issue with RAN resources, for example). The mechanism can also be used to enrich the tools that are already available to better handle attack traffic close to the source {{?RFC9066}}.

* Discovery of intentional policy applied on ACs (peering links, CE-PE links, etc.) when such information is not made available during the service activation or when network upgrades are performed.

* A user may configure policies on the CPE such as securing some resources to a specific internal host used for gaming or video streaming. The CPE can use the NRLP option to share these rate-limit policies to connected hosts to adjust their forwarding behavior.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the terms defined in {{Section 2 of ?I-D.ietf-opsawg-ntw-attachment-circuit}} and {{?RFC9473}}.

Also, this document makes use fo the following terms:

Reactive policy:
:  Treatment given to a flow when an exceptional event
   occurs, such as diminished throughput to the host caused by radio
   interference or weak radio signal, congestion on the network
   caused by other users or other applications on the same host.

Intentional policy:
:  Configured bandwidth, pps, or similar throughput
   constraints applied to a flow, application, host, or subscriber.

Rate-limit:
: Used as a generic term to refer to a policy to restrict the maximum bitrate of a flow.
: It can be used with or without any traffic classification.

# NRLP Blob {#sec-blob}

This section defines the set of attributes that are included in an NRLP blob:

Optional Parameter Flags (OPF):
: These flags indicate the presence of some optional parameters. The following flags are defined (from MSB to LSB):

    E:
    : When set to "1", this flag indicates the presence of Excess Information Rate (EIR).
    : When set to "0", this flag indicates that EIR is not present.

    P:
    : When set to "1", this flag indicates the presence of Peak Information Rate (PIR).
    : When set to "0", this flag indicates that PIR is not present.

    U:
    : Unassigned bits. See {{sec-iana-opf}}.
    : Unassigned bits MUST be set to zero by senders and MUST be ignored by receivers.

Flow flags (FF):
: These flags are used to express some generic properties of the flow. The following flags are defined (from MSB to LSB):

    S (Scope):
    : 1-bit field which specifies whether the policy is per host (when set to "1") or per subscriber (when set to "0).

    D (Direction):
    : 1-bit flag which indicates the direction on which to apply the enclosed policy.
    : When set to "1", this flag indicates that this policy is for
      network-to-host direction.
    : When set to "0", this flag indicates that this policy is for
      host-to-network direction.

    R (Reliablity):
    : 2-bit flag which indicates the reliability type of traffic on which to apply the enclosed policy.
    : When set to "00b", this flag indicates that this policy is for unreliable traffic.
    : When set to "01b", this flag indicates that this policy is for reliable traffic.
    : When set to "10b", this flag indicates that this policy is for both reliable and unreliable traffic.
    : No meaning is associated with setting the field to "11b". Such value MUST be silently ignored by the receiver.

    U:
    : Unassigned bits. See {{sec-iana-ff}}.
    :  Unassigned bits MUST be set to zero by senders and MUST be ignored by receivers.

TC (Traffic Category):
: 6-bit field which specifies a traffic category to which this policy applies.
: The following values are supported:

  + "0": All traffic. This is the default value.
  + "1": Streaming
  + "2": Real-time
  + "3": Bulk traffic
  + 4-63: Unassigned values

Committed Information Rate (CIR) (Mbps):
: Specifies the maximum number of bits that a network can receive or
  send during one second over an AC for a
  traffic category.
: If set to 0, this indicates to the host that an alternate path (if any) should be preferred over this one.
: This parameter is mandatory.

Committed Burst Size (CBS) (bytes):
: Specifies the maximum burst size that can be transmitted at CIR.
: MUST be greater than zero.
: This parameter is mandatory.

Excess Information Rate (EIR) (Mbps):
: MUST be present only if the E flag is set to '1'.
: Specifies the maximum number of bits that a network can receive or
  send during one second over an AC for a
  traffic category that is out of profile.
: This parameter is optional.

Excess Burst Size (EBS) (bytes):
: MUST be present only if EIR is also present.
: Indicates the maximum excess burst size that is allowed while not complying with the CIR.
: MUST be greater than zero, if present.
: This parameter is optional.

Peak Information Rate (PIR) (Mbps):
: MUST be present only if P flag is set to '1'.
: Traffic that exceeds the CIR and the CBS is metered to the PIR.
: This parameter is optional.

Peak Burst Size (PBS) (bytes):
: MUST be present only if PIR is also present.
: Specifies the maximum burst size that can be transmitted at PIR.
: MUST be greater than zero, if present.

The reader should refer to {{?RFC2697}}, {{?RFC2698}}, and {{?RFC4115}} for examples
of how various combinations of CIR/CBS/EIR/EBS/PIR/PBS are used for policing. Typically:

* A Single-Rate, Three-Color Marker {{?RFC2697}} uses CIR, CBS, and EBS.
* A Dual-Rate, Three-Color Marker {{?RFC2698}} uses CIR, CBS, PIR, and PBS.

# Security Considerations

An attacker who has access to the XXXX exchanged over an AC may:

Decrease the bitrate:
: This may lower the perceived QoS if the host aggressively lowers its transmission rate.

Increase the bitrate value:
: The AC will be overloaded, but still the rate-limit at the network will discard excess traffic.


# IANA Considerations

## Rate-Limit Policy Objects Registry Group {#sec-iana-rlp}

This document requests IANA to create a new registry group entitled "Rate-Limit Policy Objects".

## Optional Parameter Flags Registry {#sec-iana-opf}

This document requests IANA to create a new registry entitled "Optional Parameter Flags" under the "Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-op-flags}}.

|Bit Position|     Description|     Reference|
|1| E-flag|This-Document|
|2| P-flag|This-Document|
|3| Unassigned| |
|4| Unassigned| |
{: #iana-op-flags title="Optional Parameter Flags"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

## Flow Flags Registry {#sec-iana-ff}

This document requests IANA to create a new registry entitled "Flow flags" under the "Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-flow-flags}}.

|Bit Position|     Description|     Reference|
|1| Scope (S) Flag|This-Document|
|2| Direction (D) Flag|This-Document|
|3-4| Reliability (R) Flags|This-Document|
|5| Unassigned| |
|6| Unassigned| |
{: #iana-flow-flags title="Flow flags"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

--- back

# Acknowledgments
{:numbered="false"}

TBC.
