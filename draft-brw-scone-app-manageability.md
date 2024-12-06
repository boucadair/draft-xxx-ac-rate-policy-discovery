---
title: "SCONE Applicability & Manageability"
abbrev: "SCONE Applicability & Manageability"
category: std

docname: draft-brw-scone-app-manageability-latest
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
    fullname: Gyan Mishra
    organization: Verizon Inc
    country: United States of America
    email: "gyan.s.mishra@verizon.com"
 -
    ins: M. Amend
    name: Markus Amend
    org: Deutsche Telekom
    country: Germany
    email: markus.amend@telekom.de

 -
    ins: L. Contreras
    name: Luis M. Contreras
    org: Telefonica
    country: Spain
    email: luismiguel.contrerasmurillo@telefonica.com

normative:

informative:
     IANA-PVD:
        title: Provisioning Domains (PvDs)
        author:
        -
          organization: "IANA"
        target: https://www.iana.org/assignments/pvds/
        date: false

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

TBC

--- middle

# Introduction

{{!I-D.brw-scone-throughput-advice-blob}} defines a set of parameters that can be used by networks to share the rate-limit policies applied on a network attachment: Throughput Advice. These parameters are independent of the channel that is used by hosts to discover such policies.

Applications will have access to the Throughput Advices and will, thus,
   adjust their behavior as a function of scope and traffic category
   indicated in a policy (all traffic, etc.).

{{!I-D.brw-scone-throughput-advice-blob}} does not assume nor preclude that other mechanisms,
   e.g., Low Latency, Low Loss, and Scalable Throughput (L4S) {{?RFC9330}},
   are enabled in a bottleneck link.  The reader may refer to {{sec-alt}}
   for a list of relevant mechanisms.  Whether these mechanism as
   alternative or complementary to explicit host/network signals is to
   be further assessed.

This document discusses focuses on manageability and deployment considerations of Throughput Advice.

Operational considerations are discussed in {{sec-ops}}, while
   deployment incentives are described in {{sec-inc}}.

# Operational Considerations {#sec-ops}

## Throughput Advice Is Complementary Not Replacement Solution

Sharing Throughput Advice are not intended to replace usual actions to soften bottlenck issues (e.g., adequate network dimensioning and upgrades). However, given that such actions may not be always immediately possible or economically justified, Throughput Advices can be considered as complementary mitigations to soften these issues by introducing some collaboration between a host and
its networks to adjust their behaviors.

## Provisionning Policies

Throughput Advice senders should be configured with instructions about the type of network rate-limit policies to be shared with requesting hosts. These types can be provided using mechanisms such as {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

## Redundant vs. Useful Signal

In contexts where the bitrate policies are known during the establishment of the underlying bearer (e.g., GBR PDU Sessions), sending Throughput Advice over the AC may be redundant and should thus be disabled.

In contexts where the (average) bitrate policies provided during the establishment of a bearer cannot be refreshed to echo network-specific conditions (e.g., overload) using bearer-specific mechanisms, sending Throughput Advices over the AC would allow control the load at the source.

When both bearer-specific policies and Throughput Advices are communicated to a host, the Throughput Advices takes precedence.

## Fairness

Rate-limit policies enforced at the network are assumed to be consistent with the local jurisdictions. For example:

* {{BEREC}} states that ISPs are prohibited from blocking or slowing down of Internet traffic, except for legal reasons, network security, or congestion, provided that equivalent categories of traffic are treated equally.
* {{FCC}} states that net neutrality policies "prohibits internet service providers from blocking, throttling, or engaging in paid prioritization of lawful content". The FCC allows some exceptions, like for security and emergencies.

These regulatory frameworks align with the goals of this document.

## Architectural Considerations Matter

Approaches based on middleboxes are generally not recommended due to their inherent limitations, in terms of performance, scalability, redundancy, etc. Specifically, if the management and operation of such middleboxes remain unclear, that motivate operational issues and responsibilities.
Furthermore, it is important to note that any middlebox could not necessarily cover an entire service end-to-end, thus **producing only partial observations which could not be sufficiently good at the time of generating appropriate signals**.


## Service Considerations: Application Diversity & Realistic Assessment

Signals could be generated for multiple services and/or applications. For instance, services providing short video content might require signals different to those based on long videos. This implies the need of defining a generic method suitable for any kind of service and application, avoiding the multiplicity of solutions and the dominance of some applications over others.

It should be also noted that more experimentation is needed in order to fully understand the implications of the signals in the overall performance of the network. On one hand, the co-existence of multiple flows, some of them using the signals for improving the experience, some others not. For this, more experimentation and datasets are required, so then can be clear that no flows are negatively impacted at all.

On the other hand, if the experience of the flows improves and depending on the nature of the signals themselves, this might motivate a more intense usage of the network, then requiring to accommodate larger number of flows, and in consequence, reducing the available resources per application. This kind of paradox can be **assessed with more experimental results under realistic conditions (i.e., multiple users and multiple services in the network)**.

## Operational Guidance for Signal Enforcement

Signals are conceived as indications from the network towards applications. It is not clear the way of enforcing the application to follow the indication, especially in a context where different applications from a user, or multiple users, simultaneously access the network. This can motivate a wastage of resources for generating signals with the risk of not being effective. Furthermore, it might lead to a continuous loop of signal generation because the initial signals being ignored. It is then necessary to define mechanisms to avoid permanent signal generation when ignored.

Finally, signals could not be required at every moment, but only in situations that can benefit the service. Such situations could be due, for instance, to given levels of congestion, or based on previous information shared by the application (e.g., SLO thresholds) so that signals can be triggered according to service conditions. **Elaborating more operational guidance on intended signal enforcment policy is key**.

## Signal Estimation

The validity of the estimation produced by a network might be questioned by the application. Trust is required in a way that applications can safely follow guidance from a network. Furthermore, whatever estimation should be timely produced, avoiding the generation of aged estimations that could not correspond to the actual service circumstances. Finally, some common guidance is necessary to define a standard way of generating signals, for instance, per-flow or per group of flows.

An open point is how to deal with adaptive applications, in the sense that signals could not be of value because the self-adaptation nature of these applications.

## Signal "Interference"

The network is built on multiple layers. In some cases, different solutions targeting similar objectives (e.g., congestion control or bottleneck mitigation) can be in place. It is then necessary to **assess the simultaneous coexistence of these solutions to avoid contradictory effects or "interferences"**.

# Deployment Incentives {#sec-inc}

## Networks

There are a set of tradeoffs for networks to deploy Throughput Advice discovery:

* Cost vs. benefit
* Impact on operations vs incentive to deploy
* Enhanced experience vs. impacts on nominal mode

The procedure defined in the document provides a mechanism to assist networks managing the load at the source and, thus, contribute to better handle network overloads and optimize the use
of resources under non nominal conditions. The mechanism also allows to enhance the quality of experience at the LAN by providing a simple tool to communicate local policies to hosts. A minimal change is required to that aim.

With the OS support, the following benefits might be considered by networks:

Improved Network Performance:
: The OS can schedule network requests more efficiently, preventing network congestion, and improving overall stability and network performance with Throughput Advices.

Cost Efficiency:
: By managing network usage based on known rate limits, the OS can help reduce network-related costs. However, this is difficult to assess.

Networks that throttle bandwidth for reasons that are not compliant with local jurisdictions, not communicated to customers, etc. are unlikely to share Throughput Advices. If these signals are shared, it is unlikely that they will mirror the actual network configuration (e.g., application-specific policies).

## Applications {#sec-app-inc}

Some applications support some forms of bandwidth measurements (e.g., {{app-measurement}}) which feed
how the content is accessed to using ABR. Complementing or replacing these measurements with explicit signals
depends upon:

* The extra cost that is required to support both mechanisms at the application layer.
* The complexity balance between performing the measurements vs. consuming the signal.
* Whether the measurements ("assessed property" per {{?RFC9473}}) reflect actual network conditions or severely diverge.
* The availability of the network signals at the first place: it is unlikely that all networks will support sending the signals. Deployment incentives at the network may vary.
* The host support may be variable.

Applications that don't support (embedded) bandwidth measurement schemes will be enriched with the Throughput Advices as this will be exposed by an OS API.

## Host OS

API to facilitate Application Development:
: An OS can provide more accurate available bandwidth to applications through the API (as mentioned in {{sec-app-inc}}), making implementation easier for applications that don't requrie dedicated bandwidth measurement.

Prevent Abuse:
: The OS can allocate network resources more fairly among different processes, with Throughput Advices, ensuring that no single process monopolizes the network.

Better Resource Management:
: OS can also optimize resource allocation, by deprioritizing background/inactive applications in the event of high network utilization.

Enhanced Security:
: Awareness of Throughput Advices can help the OS detect and mitigate network-related security threats, such as denial-of-service (DoS) attacks.

Improved User Experience:
: By avoiding network congestion and ensuring fair resource allocation, the OS can provide a smoother, more responsive user experience.

Improved Application Development Efficiency:
: OS providing rate limits through an API (as mentioned in {{sec-app-inc}}) can provide the above listed benefits at per application level.

# Security Considerations

TBC.

# IANA Considerations

This document does not make any IANA request.

--- back


# Alternative/Complementary Mechanisms {#sec-alt}

In the event of bottlenecks in a network, there are other mechanisms that provide information or help to reserve resources. These can be used within the bottleneck network or, in some cases, across network boundaries. The following sections give examples of such mechanisms and provide background information.

## L4S {#L4S}

Low Latency, Low Loss, and Scalable Throughput (L4S) is an architecture defined in {{?RFC9330}} to avoid queuing at bottlenecks by capacity-seeking congestion controllers of senders. L4S support addresses the investigated use case of this document, which considers rate limiting, which typically involves queuing discipline at the rate limiting bottleneck. If all involved elements (UE, network, and service) support L4S, the use of Explicit Congestion Notification (ECN) provides the measure used to inform the network protocol and/or service endpoints in use of impending congestion. Congestion detection and reaction may require a few RTTs to adjust to the network forwarding conditions.

As of 3GPP Rel. 18 (5G Advanced, {{TS-23.501}}), L4S is also defined for the 5G system (5GS) and can be used by UE and its services, and for external parties of the 5GS by exposure of congestion information.

## Network Slicing {#ns}

One measure for guaranteeing resources in networks is network slicing. This is achieved by configuring certain resources like adequate QoS setup for communication streams, which are taken into account in packet schedulers along the transport path. e.g., the RAN air interface.

Network slicing is considered by 3GPP for 5G {{TS-23.501}} (an equivalent can be achieved in 4G by configuring QFI values), by IETF {{?RFC9543}} for transport networks, and by BBF {{TR-470}} for wireline access. A realization model in transport networks is detailed in {{?I-D.ietf-teas-5g-ns-ip-mpls}}.

L4S {{L4S}} can be used for the realization of a network slice. Network slices properties (e.g., throughput) can be retrieved from an operator network or configured by third parties via a network API {{network_api}} (e.g., 3GPP NEF).

## 3GPP UE Route Selection Policy {#ursp}

UE Route Selection Policy (URSP) is a feature specified in 3GPP to match and forward traffic based upon a selection descriptor and a route descriptor as further detailed in {{TS-23.503}}.


Specified traffic descriptors may be:

* Application
* IP
* Domain
* Non-IP
* DNN
* Connection Capabilities
* Connectivity Group ID

Specified route selection descriptors: must contain PDU Session Type Selection (e.g., IPv4v6 or IPv6) and may contain the following:


* SSC Mode
* Network Slice
* DNN
* Non-Seamless Offload indication
* Access Type preference

URSP rules that contain both descriptors can be announced from the provider network to a UE or preconfigured in the UE, possibly subscription-based. These rules can be used to identify services in the UE and to provide routes with explicit  characteristics. URSP rules might also be triggered by the usage of network APIs {{network_api}} and combined with network slicing {{ns}}, for example.

## Network APIs {#network_api}

Network APIs are the interface between the operator network and third-party providers. With 4G, the first methods were introduced to make network capabilities available, which has been greatly improved with the introduction of 5G. To this end, the new Network Exposure Function (NEF) is responsible for 5G, which is specified in {{TS-29.522}}, which defines a huge list of network capabilites for monitoring and configuration for external consumption.

For integration into external services, initiatives such as the CAMARA Alliance and GSMA Open Gateway provide abstractions of these exposed network capabilities into service APIs for easy integration by developers.

The CAMARA API "Network Slice Booking", which is currently under development, would be a way for a service provider to configure the necessary resources in the operator network. In the background, 5G features such as network slicing {{ns}} , URSP {{ursp}} and, if necessary, L4S {{L4S}} could then ensure implementation in the operator network.

# Acknowledgments
{:numbered="false"}

TBC.
