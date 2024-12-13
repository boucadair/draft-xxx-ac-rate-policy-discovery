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
  VPP:
     title: Policing
     author:
     -
       organization: "Vector Packet Processor (VPP)"
     target: https://s3-docs.fd.io/vpp/23.06/developer/corefeatures/policer.html
     date: false

--- abstract

Traffic exchanged over a network may be subject to rate-limit policies for various operational reasons.
This document specifies a generic object (called, Throughput Advice) that can be used by mechanisms for hosts
to dynamically discover these network rate-limit policies. This information is then
passed to applications that might adjust their behaviors accordingly.

The design of the throughput advice object is independent of the discovery channel (protocol, API, etc.).

--- middle

# Introduction

Connectivity services are provided by networks to customers via
dedicated terminating points, such as customer edges (CEs) or User Equipment (UE).
To facilitate data transfer via the provider network, it is assumed that appropriate setup
is provisioned over the links that connect customer terminating points and a provider network (usually via a Provider Edge (PE)),
successfully allowing data exchange over these links. The required setup is referred to in this document as network attachments,
while the underlying link is referred to as "bearers".

The bearer can be a physical or logical link that connects a customer device to a provider network. A bearer can be a wireless or wired link. The same or multiple bearer technologies can be used to establish the bearer (e.g., WLAN or cellular) to graft customer terminating points to a network.

{{ac}} shows an example of a network that connects CEs and hosts (UE, for example). These CEs are servicing
other (internal) hosts. The identification of these hosts is hidden from the network. The policies enforced at the network
for a network attachment are per-subscriber, not per-host. Typically, if a CE is provided with a /56 IPv6 prefix, policies are enforced
in the network on that /56 not the individual /64s that will be used by internal hosts. A customer terminating point may be serviced with one (e.g., UE#1, CE#1, and CE#3) or multiple network attachments (e.g., CE#2). For the sake of simplicity, {{ac}} does not show the interconnection with other networks or multi-homed CEs.

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
able to send and receive traffic over a network attachment. The required set of parameters to provision a network attachment is a function of the connectivity service offering. For example, a very limited set of parameters is required for mass-market service offering while a more elaborated set is required for Enterprise services. A comprehensive list of provisioning parameters that are available on
the PE-side of a network attachment is specified in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

As discussed, e.g., in {{Section 4.2 of ?RFC7567}}, packet dropping by network devices occurs
mainly to protect the network (e.g., congestion-unresponsive flows) and also to ensure fairness over a shared link. These policies may be intentional policies (e.g., enforced as part of the activation
of the network attachment and typically agreed upon service subscription)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation). Rate-limits are usually
configured in (ingress) nodes. These rate-limits can be shared with customers when subscribing to a connectivity service (e.g., "A YANG Data Model for Layer 2 Virtual Private Network (L2VPN) Service Delivery" {{?RFC8466}}).

{{sec-blob}} defines a set of parameters that can be used by networks to share the rate-limit policies applied on a network attachment: Throughput Advice. The set of parameters are independent of the address family.

This document does not assume nor preclude any specific signaling protocol to share the throughput advices. These parameters are independent of the channel that is used by hosts to discover such policies.

Whether host-to-network, network-to-host, or both policies are included in throughput advice is deployment specific. All these combinations are supported in this document.

Also, one or more throughput advice instances may be returned for a given traffic direction. Examples of such instances are discussed in {{sec-ex}}.

As one can infer from the name, a throughput advice is advisory in nature. The advice is provided solely as a hint.

In order to ease mapping with specific signaling mechanisms, allow for future extensions, and ensure consistent use of the advice, a new IANA registry is created in {{sec-iana}}.

# What's Out?

This document does not make any assumption about:

* The type of network (fixed, cellular, etc.) that terminates a network attachment.
* The services or applications that are delivered over a network attachment. Whether one or multiple services
are bound to the same network attachment is deployment specific.
* How the throughput advice is computed/set.
* The protocol machinery for validating, refreshing, detecting stale, and flushing out received advices.
* How applications running over a host can learn the bitrates associated with a network attachment. Typically, this can be achieved by invoking a dedicated API. However, the exact details of the API(s) is OS-specific and, thus, out of scope of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document makes use of the following term:

Rate-limit:
: Used as a generic term to refer to a policy to restrict the maximum bitrate over a network attachment.
: It can be used with or without any traffic classification.
: A rate-limit can involve limiting the rate and/or burst size.

# Sample Deployment Cases {#sec-uc}

Some deployment use cases for throughput advice discovery are provided below:

Adaptive Application Behavior:
: Discovery of intentional policy applied on network attachments when such information is not made available during the service activation or when network upgrades are performed. Adaptive applications will use the information to adjust their behavior.
: Concretely, applications are supposed to have access to all throughput advice instances and would, thus, adjust their behavior as a function of the parameters indicated in a throughput policy.
: Likewise, a host with multiple network attachments may use the discovered throughput advice instances over each network attachment to decide how to distribute its flows over these network attachments (prefer a network attachment to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policy.
: The throughput advice can feed mechanisms such as {{Section 4.4.2 of ?RFC7661}} or {{Section 7.8 of ?RFC9002}} to control the maximum burst size.

Network Assisted Offload:
: A network may advertize a throughput advice when it is overloaded, including when it is under attack. The rate-limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events (issue with RAN resources, for example).
: The mechanism can also be used to enrich the tools that are already available to better handle attack traffic close to the source {{?RFC9066}}.

Better Local Services:
: A user may configure policies on the CE such as securing some resources to a specific internal host used, e.g., for gaming or video streaming. The CE can use the throughput advice to share these rate-limit policies to connected hosts to adjust their forwarding behavior. Controlling the load at the source will allow to partition the resources between connected hosts.

# Throughput Advice Object {#sec-blob}

## Throughput Parameters

The throughput advice parameters leverage existing technologies for configuring policies in provider networks. {{sec-overview}} provides a brief overview of how inbound policies are enforced in ingress network nodes. The reader may refer to {{?RFC2697}}, {{?RFC2698}}, and {{?RFC4115}} for examples of how various combinations of Committed Information Rate (CIR), Committed Burst Size (CBS), Excess Information Rate (EIR), Excess Burst Size (EBS), Peak Information Rate (PIR), and Peak Burst Size (PBS) are used for policing. Typically:

* A Single-Rate, Two-Color Marker (1r2c) uses CIR and CBS.
* A Single-Rate, Three-Color Marker (1r3c) {{?RFC2697}} uses CIR, CBS, and EBS.
* A Dual-Rate, Three-Color Marker (2r3c) {{?RFC2698}} uses CIR, CBS, PIR, and PBS.
* 2r3c when implemented with {{?RFC4115}} uses CIR, CBS, EIR, and EBS. This mode allows for a better handling of in-profile traffic (refer to {{Section 1 of ?RFC4115}} for more details).

An implementation example of these variants (and others) can be found at {{VPP}}.

This version of the document uses the common denominator of all these policies: CIR/CBS.

## Overall Object Structure

A throughput advice object may include multiple throughput advices (referred to as "throughput advice instances"), each covering a specific match criteria. Each of these instances adheres to the structure defined in {{sec-ins-structure}}.

Throughput advice objects are bound to the network interface over which the advice was received.

The throughput advice object is described in CDDL {{!RFC8610}} format shown in {{cddl}}. This format is meant to ease mapping with encoding specifics of a given discovery channel that supplies the throughput advice.

~~~~ CDDL
; Provides information about the rate-limit policy that is
; enforced for a network attachment.
; One or more throughput instances can be present in an advice.

throughput-advice =  [+ throughput-instance]

throughput-instance =  {
  ? instance-flags => flags,
  ? traffic-category => category,
  throughput => rate-limit
}

; Indicates scope, traffic direction, and reliability type.
; Default value for scope is per subscriber policy.
; Default value for direction is network-to-host direction.
; Default value for reliability is false (i.e., the policy is
; applicable to both reliable and unreliable traffic).
; If any of these parameters is not present, this is equivalent
; to enclosing the parameter with its default value.

flags =  {
  ? scope: &scope-values .default subscriber,
  ? direction: &direction-values .default n2h,
  ? reliability: &reliability-values .default any
}

scope-values = (subscriber: 0, host: 1, flow: 2)
direction-values = (n2h: 0, h2n: 1, bidir: 2)
reliability-values = (any: 0, reliable: 1, unreliable: 2)

; Indicates traffic category to which the policy is bound.
; If the value is set to 0, this means that the policy is
; enforced for all traffic.

category =  {
  ? tc: uint .default 0
}

; Indicates the rate and burst limits.
; Only CIR/CBS are mandatory to include.

rate-limit = {
  cir: uint,          ; Mbps
  cbs: uint .gt 0,    ; bytes
}
~~~~
{: #cddl title="Throughput Advice Object Format in CDDL"}

## Throughput Advice Instance Attributes {#sec-ins-structure}

This section defines the set of attributes that are included in a throughput advice instance:

Instance Flags (IF):
: These flags are used to express some generic properties of the applicability of the instance. The following flags are defined:

    S (Scope):
    : Indicates the granularity of enforcing policies.
    : This parameter specifies whether the policy is a per-subscriber, per-host, or per-flow policy.

    D (Direction):
    : Indicates the direction on which to apply the enclosed policy.
    : When set to "00b", this flag indicates that this policy is for
      network-to-host direction.
    : When set to "01b", this flag indicates that this policy is for
      host-to-network direction.
    : When set to "10b", this flag indicates that this policy is for
      both network-to-host and host-to-network directions.

    R (Reliability):
    : Indicates the reliability type of traffic on which to apply the enclosed policy.
    : For example, Reliable could map to Queue-Building (QB) and unreliable could map to Non-Queue-Building (NQB). One of the ways for application to make reliability markings visible is by following, e.g., the considerations in {{Section 4 of ?I-D.ietf-tsvwg-nqb}}.
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
: An average rate that specifies the maximum number of bits that a network can
  send (or receive) during one second over a network attachment.
: The CIR value MUST be greater than or equal to 0.
: If set to 0 (or a very low value), this indicates to the host that alternate paths (if any) should be preferred over this one.
: This parameter is mandatory.

Committed Burst Size (CBS) (bytes):
: Specifies the maximum burst size that can be transmitted at CIR.
: MUST be greater than zero.
: This parameter is mandatory.

# Examples {#sec-ex}

For the sake of illustration, {{ex}} exemplifies the content of a throughput advice using JSON notations. The advice
includes one rate-limit instance that covers network-to-host traffic direction and is applicable to all traffic destined to any host of a subscriber.

~~~~~json
{
    "throughput-advice": [
        {
            "direction": 0,
            "scope": 0,
            "tc": 0,
            "cir": 50,
            "cbs": 10000
        }
    ]
}
~~~~~
{: #ex title="A JSON Example"}

The advice conveyed in {{ex-2}} is similar to the advice in {{ex}}. The only difference is that default values are not explicitly signaled in {{ex-2}}.

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
{: #ex-2 title="A JSON Example with Default Values Not Explicitly Signaled"}

{{ex-3}} shows the example of an advice that encloses two instances, each for one traffic direction.

~~~~~json
{
    "throughput-advice": [
        {
            "direction": 0,
            "cir": 50,
            "cbs": 10000
        },
        {
            "direction": 1,
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

# Security Considerations

As discussed in {{sec-uc}}, sharing a throughput advice helps networks mitigate overloads, particularly during periods of high traffic volume.

An attacker who has the ability to change the throughput advice objects exchanged over a network attachment may:

Decrease the bitrate value:
: This may lower the perceived QoS if the host aggressively lowers its transmission rate.

Increase the bitrate value:
: The network attachment will be overloaded, but still the rate-limit at the network will discard excess traffic.

Delete or remove the advice:
: This is equivalent to deployments where the advice is not shared.

# IANA Considerations {#sec-iana}

## Rate-Limit Policy Objects Registry Group {#sec-iana-rlp}

This document requests IANA to create a new registry group entitled "SCONE Rate-Limit Policy Objects".

## Instance Flags Registry {#sec-iana-ff}

This document requests IANA to create a new registry entitled "Instance flags" under the "SCONE Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-flow-flags}}.

|Bit Position|Label|     Description   |    Reference|
|1           | S   | Scope             |This-Document|
|2-3         | D   | Direction         |This-Document|
|4-5         | R   | Reliability       |This-Document|
|6-8         |     | Unassigned        |             |
{: #iana-flow-flags title="Instance Flags"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

## Traffic Category Registry {#sec-iana-tc}

This document requests IANA to create a new registry entitled "Traffic Category Types" under the "SCONE Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-tc}}.

|Value|     Description|     Reference|
|0    | All traffic    |This-Document|
|1-63 | Unassigned     |             |
{: #iana-tc title="Traffic Category Values"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

## Rate Parameters Registry {#sec-iana-rate}

This document requests IANA to create a new registry entitled "Rate Parameters" under the "SCONE Rate-Limit Policy Objects" registry group ({{sec-iana-rlp}}).

The initial values of this registry is provided in {{iana-rate}}.

|Parameter|     Description                 |Mandatory (Y/N)|     Reference|
|cir      | Committed Information Rate (CIR)|Y              |This-Document |
|cbs      | Committed Burst Size (CBS)      |Y              |This-Document |
{: #iana-rate title="Initial Rate Parameters Values Values"}

The allocation policy of this new registry is "IETF Review" ({{Section 4.8 of !RFC8126}}).

--- back

# Overview of Network Rate-Limit Policies {#sec-overview}

As discussed, for example in {{?I-D.ietf-teas-5g-ns-ip-mpls}}, a provider network's inbound policy can be implemented using one
of following options:

   *  1r2c (single-rate two-color) rate limiter

      This is the most basic rate limiter, described in {{Section 2.3 of ?RFC2475}}.
      It meters at an ingress interface a
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

Thanks to Eduard Vasilenko for the comments.
