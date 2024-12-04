---
title: "Discovery of Network Rate-Limit Policies (NRLPs)"
abbrev: "Rate-Limit Policies Discovery"
category: std

docname: draft-brw-scone-rate-policy-discovery-latest
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

     NRLP-WIRE:
        title: Examples of Wire Format Options
        target: https://github.com/boucadair/draft-xxx-ac-rate-policy-discovery/blob/main/example-nrlp-wire-format.md
        date: false

--- abstract

This document specifies mechanims for hosts to dynamically discover Network Rate-Limit Policies (NRLPs). This information is then passed to applications that might adjust their behaviors accordingly.

Networks already support mechanisms to advertize a set of network properties to hosts (e.g., link MTU (RFC 4861) and PREFIX64 (RFC 8781)). This document complements these tools and specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate NRLPs to hosts. For address family parity, a new DHCP option is also defined. The document also discusses how Provisioning Domains (PvD) can be used to notify hosts with NRLPs.

--- middle

# Introduction

To optimally deliver connectivity services via a network attachment, networks advertize a set of network properties {{?RFC9473}} to connected hosts such as:

Link Maximum Transmission Unit (MTU):
: For example, the 3GPP {{TS-23.501}} specifies that "the link MTU size for IPv4 is sent to the UE by including it in the PCO (see TS 24.501). The link MTU size for IPv6 is sent to the UE by including it in the IPv6 Router Advertisement message (see RFC 4861)".
: {{Section 2.10 of ?RFC7066}} indicates that a cellular host should honor the MTU option in the Router Advertisement ({{Section 4.6.4 of !RFC4861}}) given that the 3GPP system
architecture uses extensive tunneling in its packet core network below the 3GPP link, and this may lead to packet fragmentation issues.
: MTU is cited as an example of path properties in {{Section 4 of ?RFC9473}}.

Prefixes of Network Address and Protocol Translation from IPv6 clients to IPv4 servers (NAT64) {{?RFC8781}}:
: This option is useful to enable local DNSSEC validation, support networks with no DNS64, support IPv4 address literals on an IPv6-only host, etc.
: NAT is cited as an example of path properties (see "Service Function" bullet in {{Section 4 of ?RFC9473}}).

Traffic exchanged over a network may be subject to rate-limit policies for various operational reasons. {{!I-D.brw-scone-throughput-advice-blob}} specifies a generic object (called, Throughput Advice) that can be used by mechanims for hosts to dynamically discover these network rate-limit policies. This information can then passed to applications that might adjust their behaviors accordingly.

Given that all IPv6 hosts and networks are required to support Neighbor Discovery {{!RFC4861}}, this document specifies a Neighbor Discovery option to be used in Router Advertisements (RAs) to communicate rate-limit policies to hosts ({{sec-nd}}). The main motivations for the use of ND for such a discovery are listed in {{Section 3 of ?RFC8781}}:

* Fate sharing
* Atomic configuration
* Updatability: change the policy at any time
* Deployability

For address family parity, a DHCP option {{!RFC2132}} is also defined for IPv4 in {{sec-dhcp}}. {{sec-pvd}} describes a discovery approach using Provisioning Domains (PvDs) {{!RFC8801}}.

These options are called: Network Rate-Limit Policy (NRLP).

Whether host-to-network, network-to-host, or both policies are returned in an NRLP is deployment specific. All these combinations are supported in this document. Also, the design supports returning one more NRLP instances for a given traffic direction.

Applications will have access to all available NRLPs and will, thus, adjust their behavior as a function of scope and traffic category indicated in a policy. Likewise, a host with multiple network attachments may use the discovered NRLPs to decide how to distribute its flows over these network attachments (prefer a network attachment to place an application session, migrate connection, etc.). That's said, this document does not make any recommendation about how a receiving host uses the discovered policies.

Networks that advertize NLRPs are likely to maintain the policing in place within the network because of the trust model (hosts are not considered as trusted devices). Per-subscriber rate-limit policies are generally recommended to protect nodes against Denial of Service (DoS) attacks (e.g., {{Section 9.3 of ?RFC8803}}). Discussion about conditions under which such a trust model can be relaxed is out of scope of this document.

To enhance flexibility in applying rate-limiting policies and better accommodate diverse endpoint performance requirements, mechanisms such as solicited Router Advertisements (RAs) {{?RFC8273}} and endpoint-specific DHCP responses can be used. These unicast responses enable granular signaling of rate-limit policies to individual endpoints, facilitating differentiated rate-limit configurations. However, this document does not prescribe how resources should be partitioned within local networks, as such considerations fall outside its scope.

This document does not assume nor preclude that other mechanisms, e.g., Low Latency, Low Loss, and Scalable Throughput (L4S) {{?RFC9330}}, are enabled in a bottleneck link. The reader may refer to I-D.brw-scone-manageability for a list of relevant mechanisms.

Refer to {{NRLP-WIRE}} for configuration examples to use NRLP.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document uses the terms defined in {{!I-D.brw-scone-throughput-advice-blob}}.

# Common NLRP Parameters {#sec-common}

The following common fields are present in all NRLP options:

## Optional Parameter Flags (OPF)

The format of this 4-bit flags is shown in {{opt-format-opf}}. This field indicates the presence of some optional inforamtion in the option.

~~~~
 0 1 2 3
+-+-+-+-+
|U|U|P|E|
+-+-+-+-+
~~~~
{: #opt-format-opf title="Optional Parameter Flags Field" artwork-align="center"}

See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}} for the meaning of the P/E flags.

U are unassigned bits. These bits MUST be set to zero by senders and MUST be ignored by receivers.

##  Flow flags (FF)

The format of this 6-bit flags is shown in {{opt-format-ff}}. This field is used to express some generic properties of the flow.

~~~~
 0 1 2 3 4 5
+-+-+-+-+-+-+
|U|R|R|D|D|S|
+-+-+-+-+-+-+
~~~~
{: #opt-format-ff title="Flow flags Field" artwork-align="center"}

See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}} for the meaning of the R/D/S flags.

U are unassigned bits. These bits MUST be set to zero by senders and MUST be ignored by receivers.

## Traffic Category (TC) & Throughput Parameters

The following parameters are used:

TC:
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}.

Committed Information Rate (CIR) (Mbps):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}.
: This is a mandatory parameter.

Committed Burst Size (CBS) (bytes):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}.
: This is a mandatory parameter.

Excess Information Rate (EIR) (Mbps):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}. This is an optional field.

Excess Burst Size (EBS) (bytes):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}. This is an optional field.

Peak Information Rate (PIR) (Mbps):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}. This is an optional field.

Peak Burst Size (PBS) (bytes):
: See {{Section 5 of !I-D.brw-scone-throughput-advice-blob}}. This is an optional field.

# IPv6 RA NRLP Option {#sec-nd}

## Option Format {#sec-ndf}

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

Refer to {{sec-common}} for the meaning of the other parameters.

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

# DHCP NRLP Option {#sec-dhcp}

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

Refer to {{sec-common}} for the meaning of the other parameters.

OPTION_V4_NRLP is a concatenation-requiring option. As such, the mechanism specified in {{!RFC3396}} MUST be used if OPTION_V4_NRLP exceeds the maximum DHCP option size of 255 octets.

OPTION_V4_NRLP is permitted to be included in the RADIUS DHCPv4-Options Attribute {{!RFC9445}}.

## DHCPv4 Client Behavior

To discover a network rate-limit policy, the DHCP client includes OPTION_V4_NRLP in a Parameter Request List option {{!RFC2132}}.

The DHCP client MUST be prepared to receive multiple "NRLP Instance Data" field entries in the OPTION_V4_NRLP option; each instance is to be treated as a separate network rate-limit policy.

# Provisioning Domains {#sec-pvd}

PvD may also be used as a mechanism to discover NRLP. Typically, the network will configured to set the H-flag so clients can
request PvD Additional Information ({{Section 4.1 of !RFC8801}}).

{{pvd-ex}} provides an example of the returned "application/pvd+json" to indicate a network-to-host
NRLP for all subscriber traffic. The NRLP list may include multiple instances if distinct policies
are to be returned for distinct traffic categories.

~~~~~json
{
   "nrlp":[
      {
         "direction":0,
         "scope":0,
         "tc":0,
         "cir":50,
         "cbs":10000,
         "ebs":20000
      }
   ]
}
~~~~~
{: #pvd-ex title="NRLP Example with PvD"}

# Security Considerations

The techniques discussed in the document offer the following security benefit: An OS can identify the type of application (background, foreground, streaming, real-time, etc.) and enforce appropriate network policies, even if a misbehaving application tries
to evade the rate-limit policies. If an application attempts to bypass rate-limiting by changing its 5-tuple or creating multiple flows,
the OS can detect this and manage the application's traffic accordingly.

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

## Neighbor Discovery Option {#sec-iana-ra}

This document requests IANA to assign the following new IPv6 Neighbor Discovery Option
type in the "IPv6 Neighbor Discovery Option Formats" sub-registry under the "Internet Control Message Protocol version 6 (ICMPv6)
Parameters" registry maintained at {{IANA-ND}}.

|Type| Description| Reference    |
|TBD1| NRLP Option| This-Document|
{: #iana-new-op title="Neighbor Discovery NRLP Option"}

> Note to the RFC Editor: Please replace all "TBD1" occurrences with the assigned value.

## DHCP Option {#sec-iana-dhcp}

This document requests IANA to assign the following new DHCP Option Code in the "BOOTP Vendor Extensions and DHCP Options" registry maintained at {{IANA-BOOTP}}.

|Tag |     Name     | Data Length| Meaning   |Reference    |
|TBD2|OPTION_V4_NRLP|N           |NRLP Option|This-Document|
{: #iana-new-dhcp title="DHCP NRLP Option"}

> Note to the RFC Editor: Please replace all "TBD2" occurrences with the assigned value.

## DHCP Options Permitted in the RADIUS DHCPv4-Options Attribute

This document requests IANA to add the following DHCP Option Code to the "DHCP Options Permitted in the RADIUS DHCPv4-Options Attribute" registry maintained at {{IANA-BOOTP}}.

|Tag |     Name     |    Reference|
|TBD2|OPTION_V4_NRLP|This-Document|
{: #iana-radius-dhcp title="New DHCP Option Permitted in the RADIUS DHCPv4-Options Attribute Registry"}

## Provisioning Domains Split DNS Additional Information

IANA is requested to add the following entry to the "Additional Information PvD Keys"
registry under the "Provisioning Domains (PvDs)" registry group {{IANA-PVD}}:

JSON key:
: "nrlp"

Description:
: "Network Rate-Limit Policies (NRLPs)"

Type:
: Array of Objects

Example:

~~~~
   {
      "nrlp":[
         {
            "direction":0,
            "scope":0,
            "tc":0,
            "cir":50,
            "cbs": 10000
         }
      ]
   }
~~~~

Reference:
: This-Document

## New PvD Network Rate-Limit Policies (NRLPs) Registry

IANA is requested to create a new registry "PvD Rate-Limit Policies (NRLPs)" registry,
within the "Provisioning Domains (PvDs)" registry group.

The initial contents of this registry are provided in {{iana-pvd-initial}}.

| JSON key   | Description           | Type    | Example         | Reference |
|direction   |Indicates the traffic direction to which a policy applies|integer|1 |This-Document|
|scope|Specifies whether the policy is per host (when set to "1") or per subscriber (when set to "0)|Boolean|1 |This-Document|
|tc|Specifies a traffic category to which this policy applies|Integer|0|This-Document|
|cir|Committed Information Rate (CIR)|Integer|50|This-Document|
|cbs|Committed Burst Size (CBS)|Integer|10000|This-Document|
|eir|Excess Information Rate (EIR)|Integer|30|This-Document|
|ebs|Excess  Burst Size (EBS)|Integer|5000|This-Document|
|pir|Peak Information Rate (PIR)|Integer|70|This-Document|
|pbs|Peak  Burst Size (PBS)|Integer|20000|This-Document|
{: #iana-pvd-initial title="Initial PvD Network Rate-Limit Policies (NRLPs) Registry Content"}

Assignments must not be added directly to the "PvD Network Rate-Limit Policies (NRLPs)" registry.
When a new attribute is added to the "SCONE Rate-Limit Policy Objects" Registry Group created by {{!I-D.brw-scone-throughput-advice-blob}},
a new JSON key is mirrored in the "PvD Network Rate-Limit Policies (NRLPs)" registry.

--- back

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

# Acknowledgments
{:numbered="false"}

Thanks to Tommy Pauly for the comment on PvD.
