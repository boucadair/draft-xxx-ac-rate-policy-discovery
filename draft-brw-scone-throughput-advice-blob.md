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

Traffic exchanged over a network may be subject to rate-limit policies for various operational reasons.
This document specifies a generic object that can be used by mechanims for hosts
to dynamically discover network rate-limit policies. This information is then
passed to applications that might adjust their behaviors accordingly. The design of the object is independent of
the discovery channel (protocol, API, etc.).

--- middle

# Introduction

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE).
To facilitate data transfer via the provider network, it is assumed that appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
successfully allowing data exchange over these links. The required setup is referred to in this document as network attachments,
while the underlying link is referred to as "bearers".

The bearer can be a physical or logical link that connects a customer device to a provider network. A bearer can be a wireless or wired link. The same or multiple bearer technologies can be used to establish the bearer (e.g., WLAN or cellular) to graft customer terminating points to a network.

> Network attachment is also known as "Attachment Circuit (AC)" which is an established concept in the industry and also in the IETF ({{?RFC4026}}, {{?RFC4664}}, {{?RFC4364}}, etc.).

{{ac}} shows an example of a network that connects CEs and hosts (UE, for example). These CEs are servicing
other (internal) hosts. The identification of these hosts is hidden from the network. The policies enforced at the network
for a network attachment are per-subscriber, not per-host. Typically, if a CE is provided with a /56 IPv6 prefix, policies are enforced
on that /56 not the individual /64s that will be used by internal hosts. A customer terminating point may be serviced with one (e.g., UE#1, CE#1, and CE#3) or multiple network attachments (e.g., CE#2).

> {{ac}} does not show the interconnection with other networks for the sake of simplicity.

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
the PE-side of a network attachment is specified in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

The required set of parameters to provision a network attachment is a function of the service offering. For example, a very limited set of parameters is required for mass-market service offering while a more elaborated set is required for Enterprise services.

As discussed, e.g., in {{Section 4.2 of ?RFC7567}}, packet dropping by network devices occurs
mainly to protect the network (e.g., congestion-unresponsive flows) and also to ensure fairness over a shared link. These policies may be intentional policies (e.g., enforced as part of the activation
of the network attachment and typically agreed upon service subscription)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

Rate-limits are usually configured in (ingress) nodes. These rate-limits can be shared with customers when subscribing to a connectivity service (e.g., "A YANG Data Model for Layer 2 Virtual Private Network (L2VPN) Service Deliver" {{?RFC8466}}).

{{sec-blob}} defines a set parameters that can be used by networks to share the rate-limit policies applied on a network attachment: throughput Advice. The set of parameters are independent of the address family.

This document does not assume nor preclude any specific signaling protocol to share the throughput advices. These parameters are independent of the channel that is used by hosts to discover such policies.

Whether host-to-network, network-to-host, or both policies are returned in a throughput advice is deployment specific. All these combinations are supported in this document.

Also, one more throughput advice instances may be returned for a given traffic direction. Each of these instances may cover a specific traffic category.

The document leverages existing technologies for configuring policies in provider networks. {{sec-overview}} provides a brief overview of how inbound policies are enforced in ingress network nodes. The reader may refer to {{?RFC2697}}, {{?RFC2698}}, and {{?RFC4115}} for examples
of how various combinations of Committed Information Rate (CIR), Committed Burst Size (CBS), Excess Information Rate (EIR), Excess Burst Size (EBS), Peak Information Rate (PIR), and Peak Burst Size (PBS) are used for policing. Typically:

* A Single-Rate, Three-Color Marker {{?RFC2697}} uses CIR, CBS, and EBS.
* A Dual-Rate, Three-Color Marker {{?RFC2698}} uses CIR, CBS, PIR, and PBS. Note that when implemented with {{?RFC4115}}, it allows for a better handling of in-profile traffic (refer to {{Section 1 of ?RFC4115}} for more details).

Sample uses of the advice are listed in {{sec-samples}}.

In order to ease mapping with specific signaling mechanims, allow for future extensions, and ensure consistent use of the advice, a new IANA registry is created in {{sec-iana}}.

# What's Out?

This document does not make any assumption about:

* The type of the network (fixed, cellular, etc.) that terminates a network attachment.
* The services or applications that are delivered over a network attachment. Whether one or multiple services
are bound to the same network attachment is deployment specific.
* How the throughput advice is computed/set.
* How applications running over a host can learn the bitrates associated with a network attachment. Typically, this can be achieved by invoking a dedicated API. However, the exact details of the API is OS-specific and, thus, out of scope of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document makes use fo the following term:

Rate-limit:
: Used as a generic term to refer to a policy to restrict the maximum bitrate of a flow.
: It can be used with or without any traffic classification.
: A rate-limit can involve limiting the rate and/or burst size.

# Sample Deployment Cases {#sec-uc}

Some deployment use cases for throughput advice discovery are provided below:

Adaptive Application Behavior:
: Discovery of intentional policy applied on network attachements when such information is not made available during the service activation or when network upgrades are performed. Adaptive applications will thus used the information to adjust their behavior.
: Concretely, applications are supposed to have access to all throughput advice instances and would, thus, adjust their behavior as a function of scope and traffic category indicated in a throughput policy (all traffic, streaming, etc.). An application that couples multiple flow types would adjust each flow type to be consistent with the specific policy for the relevant traffic category.
: Likewise, a host with multiple network attachments may use the discovered throughput advice instances over each network attachment to decide how to distribute its flows over these network attachments (prefer a network attachment to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policy.

Network Assisted Offload:
: A network may advertize a throughput advice when it is overloaded, including when it is under attack. The rate-limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events (issue with RAN resources, for example).
: The mechanism can also be used to enrich the tools that are already available to better handle attack traffic close to the source {{?RFC9066}}.

Better Local Services:
: A user may configure policies on the CE such as securing some resources to a specific internal host used for gaming or video streaming. The CE can use the throughput advice to share these rate-limit policies to connected hosts to adjust their forwarding behavior. Controling the load at the source will allow to partition the resources between connected hosts.

# Throughput Advice Object {#sec-blob}

## Overall Object Structure

A throughput advice object may include multiple throughput advices (referred to as "throughput advice instances"), each covering a specific match criteria. Each of these adhere to the structure defined in {{sec-ins-structure}}.

The throughput advice object is described in CDDL {{!RFC8610}} format shown in {{cddl}}. This format is meant to ease mapping with encoding specifics of a given discovery channel that supplies the throughput advice.

~~~~ CDDL
; Provides information about the rate-limit policy that is
; enforced for a network attachment.
; One or more throughput instances can be present in an advice.

throughput-advice =  [+ throughput-instance]

throughput-instance =  {
  ? optional-parameter-flags => opf,
  ? flow-flags => ff,
  ? traffic-category => category,
  throughput => rate-limit
}

; opf controls the presence of optional parameters such as
; excess and peak rates.
; Settting these parameters to false means that excess and
; peak parameters are not supplied in the policy.
; These control bits may not be required for protocols
; with built-in mechanisms to parse objects even with
; optional/variable fields.

opf =  {
  ? excess: bool .default false,
  ? peak: bool .default false
}

; ff indicates scope (per host or per subscriber), traffic
; direction, and reliability type (reliable or unreliable).
; Default value for scope is per subscriber policy.
; Default value for direction is network-to-host direction.
; Default value for reliability is false (i.e., the policy is
; applicable to both reliable and unreliable traffic).
; If any of these parameters is not present, this is equivalent
; to enclosing the parameter with its default value.

ff =  {
  ? scope: &scope-values .default subscriber,
  ? direction: &direction-values .default n2h,
  ? reliability: &reliability-values .default any
}

scope-values = (subscriber: 0, host: 1)
direction-values = (n2h: 0, h2n: 1, bidir: 2)
reliability-values = (any: 0, reliable: 1, unreliable: 2)

; category indicates traffic category to which the policy is bound.
; If the value is set to 0, this means that the policy is
; enforced for all traffic.

category =  {
  ? tc: uint .default 0
}

; rate-limit indicates various rates (committed, excess, and peak).
; Only CIR/CBS are mandatory to include.

rate-limit =  {
  cir: uint,          ; Mbps
  cbs: uint .gt 0,    ; bytes
  ? eir: uint,        ; Mbps
  ? ebs: uint .gt 0,  ; bytes
  ? pir: uint,        ; Mbps
  ? pbs: uint .gt 0,  ; bytes
}
~~~~
{: #cddl title="Throughput Advice Object Format in CDDL"}

## Structure of a Throughput Advice Instance {#sec-ins-structure}

This section defines the set of attributes that are included in a throughput advice instance:

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

Flow flags (FF):
: These flags are used to express some generic properties of the flow. The following flags are defined (from MSB to LSB):

    S (Scope):
    : Specifies whether the policy is per host (when set to "1") or per subscriber (when set to "0).

    D (Direction):
    : Indicates the direction on which to apply the enclosed policy.
    : When set to "00b", this flag indicates that this policy is for
      network-to-host direction.
    : When set to "01b", this flag indicates that this policy is for
      host-to-network direction.
    : When set to "10b", this flag indicates that this policy is for
      both network-to-host and host-to-network directions.

    R (Reliablity):
    : Indicates the reliability type of traffic on which to apply the enclosed policy.
    : When set to "00b", this flag indicates that this policy is for both reliable and unreliable traffic.
    : When set to "01b", this flag indicates that this policy is for unreliable traffic.
    : When set to "10b", this flag indicates that this policy is for reliable traffic.
    : No meaning is associated with setting the field to "11b". Such value MUST be silently ignored by the receiver.

    U:
    : Unassigned flags. See {{sec-iana-ff}}.

TC (Traffic Category):
: Specifies a traffic category to which this policy applies.
: The following values are supported:

  + "0": All traffic. This is the default value.
  + 1-63: Unassigned values. See {{sec-iana-tc}}.

Committed Information Rate (CIR) (Mbps):
: Specifies the maximum number of bits that a network can receive or
  send during one second over a network attachment for a
  traffic category.
: If set to 0 (or a very low value), this indicates to the host that an alternate path (if any) should be preferred over this one.
: This parameter is mandatory.

Committed Burst Size (CBS) (bytes):
: Specifies the maximum burst size that can be transmitted at CIR.
: MUST be greater than zero.
: This parameter is mandatory.

Excess Information Rate (EIR) (Mbps):
: MUST be present only if the E flag is set to '1'.
: Specifies the maximum number of bits that a network can receive or
  send during one second over a network attachment for a
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
: This parameter is optional.

# Examples

For the sake of illustration, {{ex}} exemplifies the content of a throughput advice using JSON notations. The advice
includes one rate-limit instance that covers network-to-host traffic direction and is applicable to all traffic destined to any host of a subscriber.

~~~~~json
{
    "throughput-advice": [
        {
            "direction": 0,
            "scope": false,
            "tc": 0,
            "cir": 50,
            "cbs": 10000
        }
    ]
}
~~~~~
{: #ex title="A JSON Example"}

The advice conveyed in {{ex-2}} is similar to the advice in {{ex}}. The only difference is that default values are trimmed in {{ex-2}}.

~~~~~json
{
    "throughput-advice": [
        {
            "cir": 50,
            "cbs": 10000
        }
    ]
}
~~~~~
{: #ex-2 title="A JSON Example with Default Values Trimmed"}

{{ex-3}} shows the example of an advice that encloses two instances, each for one traffic direction.

~~~~~json
{
    "throughput-advice": [
        {
            "direction": 0,
            "scope": false,
            "tc": 0,
            "cir": 50,
            "cbs": 10000
        },
        {
            "direction": 1,
            "scope": false,
            "tc": 0,
            "cir": 30,
            "cbs": 8000
        }
    ]
}
~~~~~
{: #ex-3 title="A JSON Example with Both Traffic Directions"}

If both directions are covered by the same rate-limit policy, then the advice can be supplied as shown in {{ex-4}}

~~~~~json
{
    "throughput-advice": [
        {
            "direction": 2,
            "cir": 50,
            "cbs": 10000
        }
    ]
}
~~~~~
{: #ex-4 title="A JSON Example with Single Bidir Rate-Limit Policy"}

# Sample Uses of the Advice {#sec-samples}

It is out of scope of this document to make recommendations about how the advice is consumed by applications/OS/Hosts. A non-exhaustive list is provided hereafter for illustration purposes:

* Applications can send/receive data at a rate beyond the CIR up to the PIR when the network is not congested. If network feedback (e.g., packet loss or delay) indicates congestion, the application can scale back to the CIR. Otherwise, it can use the PIR for temporary throughput boosts.
* Applications can send/receive short-term bursts of data that exceed the committed burst size CBS up to the PBS if there is no congestion. This is useful for scenarios where short, high-throughput bursts are needed.
* Applications can ensure that their sending rate never exceeds the PIR and that their short-term bursts of traffic never exceeds PBS.
* The throughput advice can feed mechanisms such as {{Section 4.4.2 of ?RFC7661}} or {{Section 7.8 of ?RFC9002}} to control the maximum burst size.
* Applications can send/receive data at different rates for reliable and unreliable traffic (reliable could map to Queue-Building (QB) and unreliable could map to Non-Queue-Building (NQB)) by mapping reliability flag. One of the ways for application to make reliability markings visible is by following, e.g., the considerations in {{Section 4 of ?I-D.ietf-tsvwg-nqb}}.

# Security Considerations

The throughtput advice is bound to a subscriber, a host, and traffic category, not individual flows. This is consistent with, e.g.,
{{Section 8.1.1 of ?RFC9330}} which states that "there has never been a universal need to police the rate of individual application flows".
The rate-limits are set for various reasons (e.g., guards against resource abuse, fairness, etc.).

As discussed in {{sec-uc}}, the throughout advice assist networks to soften overloads during DDoS attacks, in paricular. Of course, other mechanisms are enabled by networks to protect against overload (e.g., DDoS mitigation {{?RFC8811}}).

An attacker who has the ability to change the throughput advice objects exchanged over a network attachment may:

Decrease the bitrate value:
: This may lower the perceived QoS if the host aggressively lowers its transmission rate.

Increase the bitrate value:
: The network attachment will be overloaded, but still the rate-limit at the network will discard excess traffic.

Delete or remove the advice:
: This is equivalent to deployments where the advice is not shared.

# IANA Considerations {#sec-iana}

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

## Traffic Category Registry {#sec-iana-tc}

This document requests IANA to create a new registry entitled "Traffic Category Types" under the "Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-tc}}.

|Value|     Description|     Reference|
|0| All traffic|This-Document|
|1-63| Unassigned| |
{: #iana-tc title="Traffic Category Values"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

--- back

# Overview of Network Rate-Limit Policies {#sec-overview}

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
      devices or hosts. 2r3c meters in general give greater flexibility for provider network edge
      enforcement regarding accepting the traffic (green),
      de-prioritizing and potentially dropping the traffic on transit during
      congestion (yellow), or hard dropping the traffic (red).

# Acknowledgments
{:numbered="false"}

TBC.
