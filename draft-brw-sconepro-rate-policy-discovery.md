---
title: "Discovery of Network Rate-Limit Policies (NRLPs)"
abbrev: "Rate-Limit Policies Discovery"
category: std

docname: draft-brw-sconepro-rate-policy-discovery-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: wit
workgroup: sconepro
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
    abbrev: DT
    country: Germany
    email: markus.amend@telekom.de


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

     TR-470:
        title: "5G Wireless Wireline Convergence Architecture - Issue 2"
        date: false
        author:
        -
          org: BBF
        target: https://www.broadband-forum.org/pdfs/tr-470-2-0-0.pdf

     RFC9330:
     RFC9543:

--- abstract

Traffic exchanged over a network attachment may be subject to rate-limit policies.
These policies may be intentional policies (e.g., enforced as part of the activation of the network attachment and typically agreed upon service subscription)
or be reactive policies (e.g., enforced temporarily to manage an overload or during a DDoS attack mitigation).

Networks already support mechanisms to advertize a set of network properties to hosts using Neighbor Discovery options. Examples of such
properties are link MTU (RFC 4861) and PREFIX64 (RFC 8781). This document complements these tools and specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. For address family parity, a new DHCP option is also defined.

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
**leverages access control, authorization, and authentication mechanisms that are already in place for the delivery of services over these ACs**. An example of an AC provided over a 3GPP network is depicted in {{ex-arch}}. It is out of the scope of this document to describe all involved components. Readers may refer to {{TS-23.501}} for more details.

{{sec-aaa}} provides another example of how existing tools can be leveraged for AAA purposes.

~~~~aasvg
  .-----.  .-----.  .-----.    .-----.  .-----.  .-----.
  |NSSF |  | NEF |  | NRF |    | PCF |  | UDM |  | AF  |
  '--+--'  '--+--'  '--+--'    '--+--'  '--+--'  '--+--'
Nnssf|    Nnef|    Nnrf|      Npcf|    Nudm|        |Naf
  ---+--------+--+-----+--+-------+---+----+--------+---
            Nausf|    Namf|       Nsmf|
              .--+--.  .--+--.     .--+------.
              │AUSR │  │ AMF │     │   SMF   │
              '-----'  '--+--'     '----+----'
                       ╱  |             |      ╲
Control Plane      N1 ╱   |N2           |N4     ╲N4
════════════════════════════════════════════════════════════
User Plane          ╱     |             │         ╲
                .---.  .-------.  N3 .--+--. N9 .--+--. N6   .--.
                |UE +--+ (R)AN +-----+ UPF +----+ UPF +-----( DN )
                '---'  '-------'     '-----'    '-----'      '--'
                   |-------AC----------|
~~~~
{: #ex-arch title="5GS Architecture" artwork-align="center"}

> The "AC" mention in {{ex-arch}} is not present in {{TS-23.501}}. It is added to the figure for the readers' convenience to position an attachment circuit.

## Networks Are Already Sharing Network Properties with Hosts

To optimally deliver connectivity services, networks also advertize a set of network properties {{?RFC9473}} to connected hosts such as:

Link Maximum Transmission Unit (MTU):
: For example, the 3GPP {{TS-23.501}} specifies that "the link MTU size for IPv4 is sent to the UE by including it in the PCO (see TS 24.501). The link MTU size for IPv6 is sent to the UE by including it in the IPv6 Router Advertisement message (see RFC 4861)".
: {{Section 2.10 of ?RFC7066}} indicates that a cellular host should honor the MTU option in the Router Advertisement ({{Section 4.6.4 of !RFC4861}}) given that the 3GPP system
architecture uses extensive tunneling in its packet core network below the 3GPP link, and this may lead to packet fragmentation issues.
: MTU is cited as an example of path properties in {{Section 4 of ?RFC9473}}.

Prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) {{?RFC8781}}:
: This option is useful to enable local DNSSEC validation, support networks with no DNS64, support IPv4 address literals on an IPv6-only host, etc.
: NAT is cited as an example of path properties (see "Service Function" bullet in {{Section 4 of ?RFC9473}}).

Encrypted DNS option {{?RFC9463}}:
: This option is used to discover encrypted DNS resolvers of a network.

## What's In?

{{?I-D.rwbr-tsvwg-signaling-use-cases}} discusses some use cases where it is beneficial to share policies with the hosts. **Given that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}**, this document specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate these policies to hosts. For address family parity, a DHCP option {{!RFC2132}} is also defined for IPv4.

These options are called: Network Rate-Limit Policy (NRLP).

This document uses the host/network metadata specified in {{Section 5.1 of !I-D.rwbr-sconepro-flow-metadata}}.

In order to ensure consistent design for both IPv4 and IPv6 ACs, {{sec-blob}} groups the set of NRLP parameters that are returned independent of the address family. This blob can be leveraged in networks where DHCP is not used and ease the mapping with specific protocols used in these networks. For example, ***a Protocol Configuration Option (PCO) {{TS-24.008}} NRLP Information Element can be defined in 3GPP***.

Whether host-to-network, network-to-host, or both policies are returned in an NRLP is deployment specific. All these combinations are supported in this document.

Also, the design supports returning one more NRLP instances for a given traffic direction.

> {{sec-pvd}} describes a candidate discovery approach using Provisioning Domains (PvDs) {{?RFC8801}}. That approach requires more discussion as PvD is not currently deployed in mobile networks.

## What's Out?

This document does not make any assumption about the type of the network (fixed, cellular, etc.) that terminates an AC.

Likewise, the document does not make any assumption about the services or applications that are delivered over an AC. Whether one or multiple services
are bound to the same AC is deployment specific.

Applications will have access to all these NRLPs and will, thus, adjust their behavior as a function of scope and traffic category indicated in a policy (all traffic, streaming, etc.). An application that couples multiple flow types will adjust each flow type to be consistent with the specific policy for the relevant traffic category. Likewise, a host with multiple ACs may use the discovered NRLPs AC to decide how to distribute its flows over these ACs (prefer an AC to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policy. Readers should refer, e.g., to {{?I-D.rwbr-tsvwg-signaling-use-cases}} for some examples.

Networks that advertize NLRPs are likely to maintain the policing in place within the network because of the trust model (hosts are not considered as trusted devices). Per-subscriber rate-limit policies are generally recommended to protect nodes against Denial of Service (DoS) attacks (e.g., {{Section 9.3 of ?RFC8803}} or {{Section 8 of ?I-D.ietf-masque-quic-proxy}}). Discussion about conditions under which such a trust model can be relaxed is out of scope of this document.

This document does not assume nor preclude that other mechanisms, e.g., Low Latency, Low Loss, and Scalable Throughput (L4S) {{?RFC9330}}, are enabled in a bottleneck link.

## Design Motivation & Rationale

The main motivations for the use of ND for such a discovery are listed in {{Section 3 of ?RFC8781}}:

* Fate sharing
* Atomic configuration
* Updatability: change the policy at any time
* Deployability

The solution specified in the document is designed to **ease integration with network management tools** that are used to manage and expose policies. It does so by leveraging the policy structure defined in {{?I-D.ietf-opsawg-ntw-attachment-circuit}}. That same structure is also used in the context of service activation such as Network Slicing {{?RFC9543}}; see the example depicted in Appendix B.5 of {{?I-D.ietf-teas-ietf-network-slice-nbi-yang}}.

The solution defined in this document:

* **Does not require any data plane change**.
* **Applicable to any transport protocol**.
* **Does not impact the connection setup delay**.
* **Does not require to reveal the identity of the target server or the application itself** to consume the signal.
* **Supports cascaded environments** where multiple levels to enforce rate-limiting polices is required (e.g., WAN and LAN shown in {{ac-casc}}). NRLP signals can be coupled or decoupled as a function of the local policy.
* **Supports signaling policies bound to one or both traffic directions**.
* Is able to **signal whether a policy applies to a specific host or all hosts of a given subscriber**.

~~~~aasvg
.------.                      .--------------------.
| Host +---+     .---.        |                    |
|  #1  |   |     |   |        |                    |
'------'   +-----+ C |        |                    |
         nrlp#2  | P +--------+      Network       |
.------.   .-----+ E | nrlp#1 |                    |
| Host |   |     |   |        |                    |
|  #2  +---'     '---'        |                    |
'------' nrlp#3               |                    |
                              '--------------------'
~~~~
{: #ac-casc title="Example of Cascaded NRLPs" artwork-align="center"}

Compared to a proxy or an encapsulation-based proposal (e.g., {{?I-D.ihlar-masque-sconepro-mediabitrate}}), the solution defined in this document:

* **Does not impact the MTU tweaking**: No packet overhead is required.
* **Does not suffer from side effects of multi-layer encryption schemes** on the packet processing and overall performance of involved network nodes. Such issues are encountered, e.g., with the tunneled mode or long header packets in the forwarded QUIC proxy mode {{?I-D.ietf-masque-quic-proxy}}.
* **Does not suffer from nested congestion control** for tunneled proxy mode.
* **Does not incur multiple round-trips** in the forwarding mode for the client to start sending Short Header packets.
* **Does not incur the overhead of unauthenticated re-encryption of QUIC packets** in the scramble transform of the forwarding mode.
* **Does not impact the forwarding peformance of network nodes**. For example, the proxy forwarded mode {{?I-D.ietf-masque-quic-proxy}} requires rewriting connection identifiers; that is, the performance degradation will be at least equivalent to NAT.
* **Does not suffer from the complications of IP address sharing {{?RFC6269}}**. Such issues are likely to be experienced for proxy-based solutions that multiplex internal connections using one or more external IP addresses.
* **Does not suffer from penalizing the proxy** which could serve both good and bad clients (e.g., launching Layer 7 DDoS attacks).
* **Does not require manipulating extra steering policies on the host** to decide which flows can be forwarded over or outside the proxy connection.
* **Requires a minor change to the network**: For IPv6, upgrade PE nodes to support a new ND option. Note that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}. For IPv4, configure DHCP servers to include a new DHCP option.


## Sample Deployment Cases

Some deployment use cases for NRLP are provided below:

* A network may advertize an NRLP when it is overloaded, including when it is under attack. The rate-limit policy is basically a reactive policy that is meant to adjust the behavior of connected hosts to better control the load during these exceptional events (issue with RAN resources, for example). The mechanism can also be used to enrich the tools that are already available to better handle attack traffic close to the source {{?RFC9066}}.

* Discovery of intentional policy applied on ACs (peering links, CE-PE links, etc.) when such information is not made available during the service activation or when network upgrades are performed.

* A user may configure policies on the CPE such as securing some resources to a specific internal host used for gaming or video streaming. The CPE can use the NRLP option to share these rate-limit policies to connected hosts to adjust their forwarding behavior.

Operational considerations are discussed in {{sec-ops}}, while deployment incentives are described in {{sec-inc}}.

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
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.
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
: See {{Section 5.1 of I-D.rwbr-sconepro-flow-metadata}}.
: This parameter is optional.

Excess Burst Size (EBS) (bytes):
: MUST be present only if EIR is also present.
: Indicates the maximum excess burst size that is allowed while not complying with the CIR.
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
MSB                                                          LSB
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Length    |  OPF  |     FF    |    TC     |
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
|     Type      |     Length    |  OPF  |     FF    |    TC     |
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
: 8-bit identifier of the NRLP option as assigned by IANA (TBD1) (see {{sec-iana-ra}}).

Length:
: 8-bit unsigned integer.  The length of the option (including
  the Type and Length fields) is in units of 8 octets.

OPF (Optional Parameter Flags):
: 4-bit flags which indicates the presence of some optional inforamtion in the option.
: See {{sec-blob}} for the structure of this field.
: See {{iana-op-flags}} for current assigned flags.

FF (Flow flags):
: 6-bit flags used to express some generic properties of the flow.
: See {{sec-blob}} for the structure of this field.
: See {{iana-flow-flags}} for current assigned flags.

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

If the receiving host has LAN capabilities (e.g., mobile CE or mobile handset with tethering), the following behavior applies:

* If an RA NRLP is advertised from the network, and absent local rate-limit policies, the
device should send RAs to the downstream attached LAN devices with the same NRLP values received from the network.

* If local rate-limit policies are provided to the device, the device may change the scope or values received from the network
to accommodate these policies. The device may decide to not relay received RAs to internal nodes if local policies were
already advertized using RAs and those policies are consistent with the network policies.

Applications running over a host can learn the bitrates associated with a network attachment by invoking a dedicated API. The exact details of the API is OS-specific and, thus, out of scope of this document.

# DHCP NRLP Option

> Note that the base DHCP can only signal a rate policy change when the
  client first joins the network or renews its lease, whereas IPv6 ND
  can update the rate policy at the network's discretion. {{?RFC6704}}
  specifies an approach for forcing reconfiguration of individual hosts
  without suffering from the limitations of the FORCERENEW design in {{?RFC3203}}.

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
: OPTION_V4_NRLP (TBD2).  (see {{sec-iana-dhcp}}).

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
|  OPF  |     FF    |    TC     |
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
|  OPF  |     FF    |    TC     |
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

OPF (Optional Parameter Flags):
: 4-bit flags which indicates the presence of some optional inforamtion in the option.
: See {{sec-blob}} for the structure of this field.
: See {{iana-op-flags}} for current assigned flags.

FF (Flow flags):
: 6-bit flags used to express some generic properties of the flow.
: See {{sec-blob}} for the structure of this field.
: See {{iana-flow-flags}} for current assigned flags.

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

OPTION_V4_NRLP is permitted to be included in the RADIUS DHCPv4-Options Attribute {{!RFC9445}}.

## DHCPv4 Client Behavior

To discover a network rate-limit policy, the DHCP client includes OPTION_V4_NRLP in a Parameter Request List option {{!RFC2132}}.

The DHCP client MUST be prepared to receive multiple "NRLP Instance Data" field entries in the OPTION_V4_NRLP option; each instance is to be treated as a separate network rate-limit policy.

# Operational Considerations {#sec-ops}

Sharing NRLP signals are not intended to replace usual actions to soften bottlenck issues (e.g., adequate network dimensioning and upgrades). However, given that such actions may not be always immediately possible or economically justified, NRLP signals can be considered as complementary mitigations to soften these issues by introducing some collaboration between a host and
its networks to adjust their behaviors.

NRLP senders should be configured with instructions about the type of network rate-limit policies to be shared with requesting hosts. These types can be provided using mechanisms such as {{?I-D.ietf-opsawg-ntw-attachment-circuit}}.

In contexts where the bitrate policies are known during the establishment of the underlying bearer (e.g., GBR PDU Sessions), sending NRLP signals over the AC may be redundant and should thus be disabled.

In contexts where the (average) bitrate policies provided during the establishment of a bearer cannot be refreshed to echo network-specific conditions (e.g., overload) using bearer-specific mechanisms, sending NRLP signals over the AC would allow control the load at the source.

When both bearer-specific policies and NRLP signals are communicated to a host, the NRLP signals takes precedence.

Rate-limit policies enforced at the network are assumed to be consistent with the local jurisdictions. For example, {{BEREC}} says the following:

> ISPs are prohibited from blocking or slowing down of Internet traffic, except where necessary. The exceptions are limited to: traffic management to comply with a legal order, to ensure network integrity and security, and to manage congestion, provided that equivalent categories of traffic are treated equally.

# Deployment Incentives {#sec-inc}

## Networks

There are a set of tradeoffs for networks to deploy NRLP discovery:

* Cost vs. benefit
* Impact on operations vs incentive to deploy
* Enhanced experience vs. impacts on nominal mode

The procedure defined in the document provides a mechanism to assist networks managing the load at the source and, thus, contribute to better handle network overloads and optimize the use
of resources under non nominal conditions. The mechanism also allows to enhance the quality of experience at the LAN by providing a simple tool to communicate local policies to hosts. A minimal change is required to that aim.

With the OS support, the following benefits might be considered by networks:

Improved Network Performance:
: The OS can schedule network requests more efficiently, preventing network congestion, and improving overall stability and network performance with NRLP signals.

Cost Efficiency:
: By managing network usage based on known rate limits, the OS can help reduce network-related costs. However, this is difficult to assess.

Networks that throttle bandwidth for reasons that are not compliant with local jurisdictions, not communicated to customers, etc. are unlikely to share NRLP signals. If these signals are shared, it is unlikely that they will mirror the actual network configuration (e.g., application-specific policies).

## Applications {#sec-app-inc}

Some applications support some forms of bandwidth measurements (e.g., {{app-measurement}}) which feed
how the content is accessed to using ABR. Complementing or replacing these measurements with explicit signals
depends upon:

* The extra cost that is required to support both mechanisms at the application layer.
* The complexity balance between performing the measurements vs. consuming the signal.
* Whether the measurements ("assessed property" per {{?RFC9473}}) reflect actual network conditions or severely diverge.
* The availability of the network signals at the first place: it is unlikely that all networks will support sending the signals. Deployment incentives at the network may vary.
* The host support may be variable.

Applications that don't support (embedded) bandwidth measurement schemes will be enriched with the NRLP signals as this will be exposed by an OS API.

## Host OS

API to facilitate Application Development:
: An OS can provide more accurate available bandwidth to applications through the API (as mentioned in {{sec-app-inc}}), making implementation easier for applications that don't requrie dedicated bandwidth measurement.

Prevent Abuse:
: The OS can allocate network resources more fairly among different processes, with NRLP signals, ensuring that no single process monopolizes the network.

Better Resource Management:
: OS can also optimize resource allocation, by deprioritizing background/inactive applications in the event of high network utilization.

Enhanced Security:
: Awareness of NRLPs can help the OS detect and mitigate network-related security threats, such as denial-of-service (DoS) attacks.

Improved User Experience:
: By avoiding network congestion and ensuring fair resource allocation, the OS can provide a smoother, more responsive user experience.

Improved Application Development Efficiency:
: OS providing rate limits through an API (as mentioned in {{sec-app-inc}}) can provide the above listed benefits at per application level.

# Security Considerations

## ND

As discussed in {{?RFC8781}}, because RAs are required in all IPv6 configuration scenarios, RAs must already be secured, e.g., by deploying an RA-Guard {{?RFC6105}}. Providing all configuration in RAs reduces the attack surface to be targeted by malicious attackers trying to provide hosts with invalid configuration, as compared to distributing the configuration through multiple different mechanisms that need to be secured independently.

RAs are already used in mobile networks to advertize the link MTU. The same security considerations for MTU discovery apply for the NRLP discover.

An attacker who has access to the RAs exchanged over an AC may:

Decrease the bitrate:
: This may lower the perceived QoS if the host aggressively lowers its transmission rate.

Increase the bitrate value:
: The AC will be overloaded, but still the rate-limit at the network will discard excess traffic.

Drop RAs:
: This is similar to the current operations, where no NRLP RA is shared.

Inject fake RAs:
: The implications are similar to the impacts of tweaking the values of a legitimate RA.

## DHCP

An attacker who has access to the DHCP exchanged over an AC may do a lot of harm (e.g., prevent access to the network).

The following mechanisms may be considered to mitigate spoofed or modified DHCP responses:

DHCPv6-Shield {{?RFC7610}}:
: The network access node (e.g., a border router, a CPE, an Access Point (AP)) discards DHCP response messages received from any local endpoint.

Source Address Validation Improvement (SAVI) solution for DHCP {{?RFC7513}}:
: The network access node filters packets with forged source IP addresses.

The above mechanisms would ensure that the endpoint receives the correct NRLP information, but these mechanisms cannot provide any information about the DHCP server or the entity hosting the DHCP server.

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

## Flow flags Registry {#sec-iana-ff}

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

## Neighbor Discovery Option {#sec-iana-ra}

This document requests IANA to assign the following new IPv6 Neighbor Discovery Option
type in the "IPv6 Neighbor Discovery Option Formats" sub-registry under the "Internet Control Message Protocol version 6 (ICMPv6)
Parameters" registry maintained at {{IANA-ND}}.

|Type|     Description|     Reference|
|TBD1|  NRLP Option|This-Document|
{: #iana-new-op title="Neighbor Discovery NRLP Option"}

> Note to the RFC Editor: Please replace all "TBD1" occurrences with the assigned value.

## DHCP Option {#sec-iana-dhcp}

This document requests IANA to assign the following new DHCP Option Code in the "BOOTP Vendor Extensions and DHCP Options" registry maintained at {{IANA-BOOTP}}.

|Tag|     Name|     Data Length|     Meaning|Reference|
|TBD2|OPTION_V4_NRLP|N|NRLP Option|This-Document|
{: #iana-new-dhcp title="DHCP NRLP Option"}

> Note to the RFC Editor: Please replace all "TBD2" occurrences with the assigned value.

## DHCP Options Permitted in the RADIUS DHCPv4-Options Attribute

This document requests IANA to add the following DHCP Option Code to the "DHCP Options Permitted in the RADIUS DHCPv4-Options Attribute" registry maintained at {{IANA-BOOTP}}.

|Tag|     Name|    |Reference|
|TBD2|OPTION_V4_NRLP|This-Document|
{: #iana-radius-dhcp title="New DHCP Option Permitted in the RADIUS DHCPv4-Options Attribute Registry"}

--- back

# Provisioning Domains {#sec-pvd}

PvD may also be considered as a mechanism to discover NRLP. Typically, the network will configured to set the H-flag so clients can
request PvD Additional Information ({{Section 4.1 of ?RFC8801}}).

{{pvd-ex}} provides an example of the returned "application/pvd+json" to indicate a network-to-host
NRLP for all subscriber traffic. The NRLP list may include multiple instances if distinct policies
are to be returned for distinct traffic categories.

~~~~~json
{
   "nrlp":[
      {
         "direction":1,
         "scope":1,
         "tc":0,
         "cir":50,
         "cbs":10000,
         "ebs":20000
      }
   ]
}
~~~~~
{: #pvd-ex title="NRLP Example with PvD"}

PvD NRLP object can be extended by defining new attributes (e.g., supply an ADN and locators of a network entity to negotiate advanced features with the network, reachability information of a network API to invoke differentiated forwarding behaviors, a slice identifier {{TS-23.501}}{{?RFC9543}}, etc.).

# Example of Authentication, Authorization, and Accounting (AAA) {#sec-aaa}

{{radius-ex}} provides an example of the exchanges that might occur between a DHCP server
and an Authentication, Authorization, and Accounting (AAA) server to retrieve the per-subscriber NRLPs.

This example assumes that the Network Access Server (NAS) embeds both Remote Authentication Dial-In User Service
(RADIUS) {{?RFC2865}} client and DHCP server capabilities.

~~~~~
   .-------------.           .-------------.             .-------.
   |    Host     |           |     NAS     |             |  AAA  |
   | DHCP Client |           | DHCP Server |             |Server |
   |             |           |RADIUS Client|             |       |
   '------+------'           '------+------'             '---+---'
          |                         |                        |
          o------DHCPDISCOVER------>|                        |
          |                         o----Access-Request ---->|
          |                         |                        |
          |                         |<----Access-Accept------o
          |                         |     DHCPv4-Options     |
          |<-----DHCPOFFER----------o    (OPTION_V4_NRLP)    |
          |     (OPTION_V4_NRLP)    |                        |
          |                         |                        |
          o-----DHCPREQUEST-------->|                        |
          |     (OPTION_V4_NRLP)    |                        |
          |                         |                        |
          |<-------DHCPACK----------o                        |
          |     (OPTION_V4_NRLP)    |                        |
          |                         |                        |

                     DHCP                    RADIUS
~~~~~
{: #radius-ex title="An Example of RADIUS NRLP Exchanges"}

# Alternative/Complementary Mechanisms

In the event of bottlenecks in a network, there are other mechanisms that provide information or help to reserve resources. These can be used within the bottleneck network or, in some cases, across network boundaries. The following sections give examples of such mechanisms and provide background information.

## L4S {#L4S}

Low Latency, Low Loss, and Scalable Throughput (L4S) is an architecture defined in {{RFC9330}} to avoid queuing at bottlenecks by capacity-seeking congestion controllers of senders. L4S support addresses the investigated use case of this document, which considers rate limiting, which typically involves queuing discipline at the rate limiting bottleneck. If all involved elements (UE, network, and service) support L4S, the use of Explicit Congestion Notification (ECN) provides the measure used to inform the network protocol and/or service endpoints in use of impending congestion. Congestion detection and reaction may require some few RTTs, though to adjust to the network forwarding conditions.

As of 3GPP Rel. 18 (5G Advanced, {{TS-23.501}}), L4S is also defined for the 5G system (5GS) and can be used by UE and its services, but also for external parties of the 5GS by exposure of congestion information.

## Network Slicing {#ns}

One measure for guaranteeing resources in networks is network slicing. This is achieved by configuring certain resources like adequate QoS setup for communication streams, which are taken into account in packet schedulers along the transport path. e.g., the RAN air interface.

Network slicing is considered by 3GPP for 5G {{TS-23.501}} (an equivalent can be achieved in 4G by configuring QFI values), by IETF {{RFC9543}} for transport networks, and by BBF {{TR-470}} for wireline access. A realization model in transport networks is detailed in {{?I-D.ietf-teas-5g-ns-ip-mpls}}.

L4S {{L4S}} can be used for the realization of a network slice. Network slices properties (e.g., throughput) can be retrieved from an operator network by third parties via a network API {{network_api}} (e.g., 3GPP NEF).

## 3GPP UE Route Selection Policy {#ursp}

UE Route Selection Policy (URSP) is a feature specified in 3GPP to match and forward traffic based upon a selection descriptor and a route descriptor as further detailed in {{TS-23.503}}.


Specified traffic descriptors may be:

* Application
* IP
* Domain
* Non-IP
* DNN
* Connection Capabilities
* PIN ID
* Connectivity Group ID

Specified route selection descriptors

* SSC Mode
* Network Slice
* DNN
* PDU Session Type
* Non-Seamless Offload indication
* Access Type preference
* ...

URSP rules that contain both descriptors can be announced from the carrier network to the UE or preconfigured in the UE, possibly subscription-based. In terms of this document, this can be used to identify services in the UE and network that are suffering from today's rate limiting policies, and to provide routes with clear characteristics or no rate limiting. URSP might also be triggered by the usage of network APIs {{network_api}} and combined with network slicing {{ns}}, for example.

## Network APIs {#network_api}

Network APIs are the interface between the operator network and third-party providers. With 4G, the first methods were introduced to make network capabilities available, which has been greatly improved with the introduction of 5G. To this end, the new Network Exposure Function (NEF) is responsible for 5G, which is specified in {{TS-29.522}}, which defines a huge list of network capabilites for monitoring and configuration for external consumption.

For integration into external services, initiatives such as the CAMARA Alliance and GSMA Open Gateway provide abstractions of these exposed network capabilities into service APIs for easy integration by developers.

The CAMARA API "Network Slice Booking", which is currently under development, would be a way for a service provider to configure the necessary resources in the operator network. In the background, 5G features such as network slicing {{ns}} , URSP {{ursp}} and, if necessary, L4S {{L4S}} could then ensure implementation in the operator network.

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
