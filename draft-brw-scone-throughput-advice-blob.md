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

--- abstract

Traffic exchanged over a network attachment may be subject to rate-limit policies.
These policies may be intentional policies (e.g., enforced as part of the activation
of the network attachment and typically agreed upon service subscription)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

This document specifies a generic object that can be be used by mechanims for hosts
to dynamically discover network rate-limit policies. This information is then
passed to applications that might adjust their behaviors accordingly. The design of the object is thus independent of
the discovery channel.

--- middle

# Introduction

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE).
To facilitate data transfer via the provider network, it is assumed that appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
successfully allowing data exchange over these links. The required setup is referred to in this document as network attachments,
while the underlying link is referred to as "bearers".

The bearer can be a physical or logical link that connects a customer device to a provider network. A bearer can be a wireless or wired link. The same or multiple bearer technologies can be used to establish the bearer (e.g., WLAN, cellular) to graft customer terminating points to a network.

> Network attachment is also known as "Attachment Circuit (AC)" which is an established concept in the industry and also in the IETF ({{?RFC4026}}, {{?RFC4664}}, {{?RFC4364}}, etc.).

{{ac}} shows an example of a network that connects CEs and hosts (UE, for example). These CEs are servicing
other (internal) hosts. The identification of these hosts is hidden from the network. The policies enforced at the network
for a network attachment are per-subscriber, not per-host. Typically, if a CE is provided with a /56 IPv6 prefix, policies are enforced
on that /56 not the individual /64s that will be used by internal hosts. A customer terminating point may be serviced with one (e.g., UE#1, CE#1, and CE#3) or multiple network attachments (e.g., CE#2). Note that the figure does not show the interconnection with other network for the sake of simplicity.

~~~~aasvg
                                                        Hosts
                                                        O O O
                                                         \|/
.------.                .--------------------.         .------.
|      +------+         |                    +---NA----+      |
| UE#1 |      |         |                    +---NA----+ CE#2 |
'------'      +---NA----+                    |         '------'
                        |     Network        |
.------.      .---NA----+                    |
|      |      |         |                    |         .------.
| CE#1 +------'         |                    +---NA----+ CE#3 |
'------'                |                    |         '------'
   /|\                  '--------------------'            /|\
  O O O                                                  O O O
  Hosts                                                  Hosts
~~~~
{: #ac title="Sample Network Attachments" artwork-align="center"}

Customer terminating points are provided with a set of information (e.g., IP address/prefix) to successfully be
able to send and receive traffic over a network attachment. A comprehensive list of provisioning parameters that are available on
the PE-side of a network attachment is documented in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

The required set of parameters to provision a network attachment is a function of the service offering. For example, a very limited set of parameters is required for mass-market service offering while a more elaborated set is required for Enterprise services (e.g., Layer 2 VPN {{?RFC9291}}).

{{sec-blob}} defines a set parameters that can be used by networks to share the rate-limit policies applied on a network attachment. The set of parameters are independent of the address family.

This document does not assume nor preclude any specific signaling protocol to share the throuput advices. These parameters are independent of the channel that is used by hosts to discover such policies.

Whether host-to-network, network-to-host, or both policies are returned in a throuput advice is deployment specific. All these combinations are supported in this document.

One more throuput advice instances may be returned for a given traffic direction.

The document leverages existing technologies for configuring policies in provider networks. {{sec-overview}} provides an overview of how inbound policies are enforced in ingress network nodes. Specifically, the reader should refer to {{?RFC2697}}, {{?RFC2698}}, and {{?RFC4115}} for examples
of how various combinations of Committed Information Rate (EIR)/Committed Burst Size (CBS)/Excess Information Rate (EIR)/Excess Burst Size (EBS)/Peak Information Rate (PIR)/Peak Burst Size (PBS) are used for policing. Typically:

* A Single-Rate, Three-Color Marker {{?RFC2697}} uses CIR, CBS, and EBS.
* A Dual-Rate, Three-Color Marker {{?RFC2698}} uses CIR, CBS, PIR, and PBS.

# What's Out?

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates a network attachment.

Likewise, the document does not make any assumption about the services or applications that are delivered over a network attachment. Whether one or multiple services
are bound to the same network attachment is deployment specific.

Applications will have access to all throuput advice instances and will, thus, adjust their behavior as a function of scope and traffic category indicated in a policy (all traffic, streaming, etc.). An application that couples multiple flow types will adjust each flow type to be consistent with the specific policy for the relevant traffic category.

Likewise, a host with multiple network attachments may use the discovered throuput advice instances over each network attachment to decide how to distribute its flows over these network attachments (prefer a network attachment to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policy.

# Sample Deployment Cases

Some deployment use cases for throuput advice discovery are provided below:

Adaptive Application Behavior:
: Discovery of intentional policy applied on network attachements (CE-PE links, peering links, etc.) when such information is not made available during the service activation or when network upgrades are performed. Adaptive applications will thus used the information to adjust their behavior.

Network Assisted Offload:
: A network may advertize a throuput advice when it is overloaded, including when it is under attack. The rate-limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events (issue with RAN resources, for example). The mechanism can also be used to enrich the tools that are already available to better handle attack traffic close to the source {{?RFC9066}}.

Better Local Services:
: A user may configure policies on the CPE such as securing some resources to a specific internal host used for gaming or video streaming. The CPE can use the throuput advice to share these rate-limit policies to connected hosts to adjust their forwarding behavior.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document makes use fo the following term:

Rate-limit:
: Used as a generic term to refer to a policy to restrict the maximum bitrate of a flow.
: It can be used with or without any traffic classification.

# Throughput Advice Object {#sec-blob}

## Multiple Throughput Advices

A throuput advice object may include multiple throughput advices (referred to as "throughput advice instance"), each covering a specific match criteria. Each of these
adhere to the structure defined in {{sec-ins-structure}}.

The throughput advice object is described in CDDL {{!RFC8610}} format shown in {{cddl}}.

~~~~ CDDL
   ; Provides information about the rate-limit policy that
   ; is enforced for a network attachment.

   throughput-advice =  [+ throughput-instance]

   throughput-instance =  {
     ? optional-parameter-flags => opf,
     ? flow-flags => ff,
     ? traffic-category => tc,
     throughput => rate-limit
   }

   ; Controls the presence of optional excess and peak
   ; rate parameters.
   ; When omitted, this is equivalent to setting these
   ; parameters to 0.
   ; Settting these parameters to 0 means that excess and
   ; peak parameters are not supplied in the policy.

   opf =  {
     ? excess: bool .default 0,
     ? peak: bool .default 0
   }

   ; Indicates scope, direction, and traffic reliability.
   ; Default value for scope is 0 (i.e., per subscriber).
   ; Default value for direction is network-to-host direction.
   ; Default value for reliability is 0 (the policy is applicable
   ; to both reliable and unreliable traffic.
   ; If any of these parameters is present, this is equivalent
   ; to enclosing the paramter with its default value.

   ff =  {
     ? scope: bool .default 0,
     ? direction: bool .default 0,
     ? reliability: uint .default 0
   }

   ; Indicates traffic category.
   ; If the value is set to 0, this means the policy is
   ; enforced for all traffic.

   tc =  {
     ? tc: uint .default 0
   }

   ; Indicates various rates (committed, excess, and peak).
   ; Only CIR is mandatory to include.

   rate-limit =  {
     cir: uint,    ; Mbps
     cbs: uint,    ; bytes
     ? eir: uint,  ; Mbps
     ? ebs: uint,  ; bytes
     ? pir: uint,  ; Mbps
     ? pbs: uint   ; bytes
   }
~~~~
{: #cddl title="Throughput Advice Object Format in CDDL"}

## Structure of a Throuput Advice Instance {#sec-ins-structure}

This section defines the set of attributes that are included in a throuput advice instance:

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
    : When set to "0", this flag indicates that this policy is for
      network-to-host direction.
    : When set to "1", this flag indicates that this policy is for
      host-to-network direction.

    R (Reliablity):
    : 2-bit flag which indicates the reliability type of traffic on which to apply the enclosed policy.
    : When set to "00b", this flag indicates that this policy is for both reliable and unreliable traffic.
    : When set to "01b", this flag indicates that this policy is for unreliable traffic.
    : When set to "10b", this flag indicates that this policy is for reliable traffic.
    : No meaning is associated with setting the field to "11b". Such value MUST be silently ignored by the receiver.

    U:
    : Unassigned bits. See {{sec-iana-ff}}.
    : Unassigned bits MUST be set to zero by senders and MUST be ignored by receivers.

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

# Security Considerations

An attacker who has access to the throuput advice objects exchanged over a network attachment may:

Decrease the bitrate:
: This may lower the perceived QoS if the host aggressively lowers its transmission rate.

Increase the bitrate value:
: The network attachment will be overloaded, but still the rate-limit at the network will discard excess traffic.


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

# Overview of Provider Network Rate-Limit Policies {#sec-overview}

As discussed, for example in {{?I-D.ietf-teas-5g-ns-ip-mpls}}, a provider network's inbound policy can be implemented using one
of following options:

   *  1r2c (single-rate two-color) rate limiter

      This is the most basic rate limiter, described in {{Section 2.3 of ?RFC2475}}.
      It meters at an ingress intreface a
      traffic stream and marks its packets as in-profile
      (below CIR being enforced) or out-of-profile (above CIR being enforced).
      In-profile packets are accepted and forwarded.  Out-of profile
      packets are either dropped right at the ingress node (hard rate limiting),
      or remarked (with different MPLS TC or DSCP TN markings) to
      signify 'this packet should be dropped in the first place, if
      there is a congestion' (soft rate limiting), depending on the
      business policy of the provider network.  In the second case, while
      packets above CIR are forwarded at an ingress node, they are subject to being
      dropped during any congestion event at any place in the provider network.

   *  2r3c (two-rate three-color) rate limiter

      This was initially defined in {{?RFC2698}}, and its improved version
      in {{?RFC4115}}.  The traffic is assigned to one of the these three
      categories:

        -  Green, for traffic under CIR

        -  Yellow, for traffic between CIR and PIR

        -  Red, for traffic above PIR

      An inbound 2r3c meter implemented with {{?RFC4115}}, compared to
      {{?RFC2698}}, is more 'customer friendly' as it doesn't impose
      outbound peak-rate shaping requirements on customer edge (CE)
      devices. 2r3c meters in general give greater flexibility for provider network edge
      enforcement regarding accepting the traffic (green),
      de-prioritizing and potentially dropping the traffic on transit during
      congestion (yellow), or hard dropping the traffic (red).

# Acknowledgments
{:numbered="false"}

TBC.
